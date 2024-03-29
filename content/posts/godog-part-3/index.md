---
title: "Intro to Godog, Part 3: Tabular and Multi-Line Data"
date: 2022-04-27
slug: "/godog-part-3"
description: Data tables, example tables, and doc strings
tags:
  - Go
  - Testing
---
As we ended the previous post saying that we were going to put the focus back on the feature file and collaborate more
with our business users, we're going to cover the use of example tables, data tables, and doc strings.

As we talk with our business users, we discuss that deposits and withdrawals can happed concurrently.  Checks written
from an account are cleared by other banks at the same time the owner makes deposits/withdrawals electronically, or
payroll direct deposits are made, or ATM transactions occur, etc. We need to make sure that when these transactions
occur concurrently, the correct balance is always maintained. With input from the business user, we come up with a list
of transactions that we want to process concurrently and then validate the expected balance. The additional scenario
added to our feature is as follows.
```gherkin
Scenario: Concurrent Deposits and withdrawals
Given I have an account with 100.00 USD
 When I process the following transations:
|type       |dollars|
|deposit    |5      |
|deposit    |10     |
|withdrawal |2      |
|withdrawal |20     |
 Then the account balance must be 93.00 USD
```

In the "I process the following transations" step, the type of the input that comes after that is DataTable. Our step
implementation can take a DataTable as an input argument, and then iterate over the rows of data when the step
executes. In this case, we will start a separate go routine to process each transaction and then validate the
account balance. The step implementation written is as follows.
```go
func (a *AccountTestState) iProcessTheFollowingTransations(table *godog.Table) error {
	transactions := []transaction{}
	// first row is header row, so skip it
	for n, row := range table.Rows {
		if n > 0 {
			if len(row.Cells) < 2 {
				return fmt.Errorf("too few columns")
			}
			units, err := strconv.Atoi(row.Cells[1].Value)
			if err != nil {
				return err
			}
			money, err := NewMoney(USD, int64(units), 0)
			if err != nil {
				return err
			}
			t := transaction{strings.ToLower(row.Cells[0].Value) == "withdrawal", money}
			transactions = append(transactions, t)
		}
	}
	wg := sync.WaitGroup{}
	wg.Add(len(transactions))
	for _, t := range transactions {
		tr := t
		go func() {
			defer wg.Done()
			if tr.isWithdrawal {
				a.account.Withdraw(tr.money)
			} else {
				a.account.Deposit(tr.money)
			}
		}()
	}
	wg.Wait()
	return nil
}
```

Similar looking to the data table are the example tables. Our business user brought up the requirement that we need to
be able to view the account balance in a variety of currencies. Having one step that took a data table as an argument and
validated multiple currencies did not seem to make sense. If the step failed, which one or ones of the requested currencies
failed? It seemed that we could almost write the same scenario multiple times, with just a different currency and
expected result in each scenario. Fortunately, rather than creating numerous scenarios that are very similar, we can
use a scenario outline.  A scenario outline is almost like a template to create new scenarios, utilizing an example table
to parameter each scenario executed based on that template. In a scenario outline, you use the "scenario outline" keyword
rather than "scenario" and rather than a literal value in your steps, you use a parameter name surrounded by angle brackets.
The parameter names come from your example table (the column headers in the first row). The example table is separated from
the scenario outline by the "Examples" keyword, but otherwise it looks the same as a data table. The scenario outline
that resulted from our conversation with our business users is as follows.
```gherkin
Scenario Outline: Opening accounts in different currencies
Given I have an account with 100.00 <currency>
 Then the account balance must convert to <dollars> USD

Examples:
|currency|dollars|
|CAD     |80     |
|CNY     |16     |
|EUR     |108    |
```

We were able to use the "currency" parameter in an existing step, and then added a new step to validate the currency
value equaled the proper amount in USD after having the standared conversion rates applied. When executing the feature,
if you look at the test output, the summary section where it prints the total number of scenarios as well as scenarios
passed and scenarios failed, you will see that each example for which the scenario outine had executed is counted as
1 scenario. So in the scenario outline above, our total scenarios executed has been increased by 3. Note that no new paramter
types are needed for scenario outlines, like we used for a step that took a data table as an argument. We should be able
to use all the same steps in a scenario outline or a regular scenario.

The last request from the business users that was included in this iteration was a validation of the address
associted with the account. This address is output as a multi-line string (not the most robust approach, but just a
simple example to illustrate a step that took a multi-line string as an argument). When utilizing this functionality,
in the feature file, the multi-line string is called a "Doc String" and has triple quotes above and below the multi-line string.
Triple back ticks (```) could also be used. The new scneario looks like this:
```gherkin
Scenario: Remittance address
Given I have a new account
 Then the remittance address must be
"""
742 Evergreen Terrace
Springfield, OR
"""
```

and the new step implementation is as follows, using DocString as the parameter type.
```go
func (a *AccountTestState) theRemittanceAddressMustBe(input *godog.DocString) error {
	if a.account.RemittanceAddress() != input.Content {
		return fmt.Errorf("expected %s but found %s", input.Content, a.account.RemittanceAddress())
	}
	return nil
}
```

The output of our feature now looks like this after adding these new scenarios:
```shell
Feature: Account Maintenance

  Scenario: New account                       # features/account_behavior.feature:7
    Given I have a new account                # account_test.go:26 -> *AccountTestState
    Then the account balance must be 0.00 USD # account_test.go:109 -> *AccountTestState

  Scenario: Deposit money into account        # features/account_behavior.feature:11
    Given I have an account with 0.00 USD     # account_test.go:30 -> *AccountTestState
    When I deposit 5.00 USD                   # account_test.go:37 -> *AccountTestState
    Then the account balance must be 5.00 USD # account_test.go:109 -> *AccountTestState

  Scenario: Withdraw money from account       # features/account_behavior.feature:16
    Given I have an account with 11.00 USD    # account_test.go:30 -> *AccountTestState
    When I withdraw 5.00 USD                  # account_test.go:46 -> *AccountTestState
    Then the account balance must be 6.00 USD # account_test.go:109 -> *AccountTestState

  Scenario: Attempt to overdraw account    # features/account_behavior.feature:21
    Given I have an account with 11.00 USD # account_test.go:30 -> *AccountTestState
    When I try to withdraw 50.00 USD       # account_test.go:55 -> *AccountTestState
    Then the transaction should error      # account_test.go:118 -> *AccountTestState

  Scenario: Concurrent Deposits and withdrawals # features/account_behavior.feature:26
    Given I have an account with 100.00 USD     # account_test.go:30 -> *AccountTestState
    When I process the following transations:   # account_test.go:71 -> *AccountTestState
      | type       | dollars |
      | deposit    | 5       |
      | deposit    | 10      |
      | withdrawal | 2       |
      | withdrawal | 20      |
    Then the account balance must be 93.00 USD  # account_test.go:109 -> *AccountTestState

  Scenario Outline: Opening accounts in different currencies # features/account_behavior.feature:36
    Given I have an account with 100.00 <currency>           # account_test.go:30 -> *AccountTestState
    Then the account balance must convert to <dollars> USD   # account_test.go:125 -> *AccountTestState

    Examples:
      | currency | dollars |
      | CAD      | 80      |
      | CNY      | 16      |
      | EUR      | 108     |

  Scenario: Remittance address          # features/account_behavior.feature:46
    Given I have a new account          # account_test.go:26 -> *AccountTestState
    Then the remittance address must be # account_test.go:153 -> *AccountTestState
      """
      742 Evergreen Terrace
      Springfield, OR
      """

9 scenarios (9 passed)
22 steps (22 passed)
8.079408ms
PASS
```

All the code from this post is available on [github](https://github.com/dumpsterfireproject/godog-examples/tree/post3).

In the next post, we'll cover filtering tests by utilizing tags as well as adding a background section to include
some common test set up prior to each scenario without using a hook.