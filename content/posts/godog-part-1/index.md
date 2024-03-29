---
title: "Intro to Godog, Part 1"
date: 2022-04-24
slug: "/godog-part-1"
description: An Introduction to Godog
tags:
  - Go
  - Testing
---
Over the next few posts, I'm going to cover the various features of [Godog](https://github.com/cucumber/godog).
In the past I've found success using behavior driven development (BDD) in projects, part of which involves collaboratively
documenting expected system behaviors using a syntax like [Gherkin](https://cucumber.io/docs/gherkin/reference/).
Taking that approach, I've used tools like [Cucumber-JVM](https://cucumber.io/docs/installation/java/) or
[Cycle](https://cyclelabs.io). Godog is the official Cucumber BDD framework for Go. It has not yet reached 1.0.0, but
already is incredibly useful. In this first post, I'll cover installation and creating your first simple tests using
some basic Gherkin syntax and the standard Go testing package. In subsequent posts, I'll cover using Backgrounds,
Scenario Outlines, the new Rule keyword (as of Gherkin version 6), hooks, and more.

To start off, let's create the new project and add Godog.
```shell
mkdir godog-examples
cd godog-examples
go mod init github.com/dumpsterfireproject/godog-example
go get github.com/cucumber/godog
```

For this [example](https://github.com/dumpsterfireproject/godog-examples/tree/post1), I'll be using an example of a
bank account. At this point, we'll just support simple functionality around creating a new account, depositing money
into the account, and withdrawing money from the account. For this first iteration, after conversations with our
(hypothetical) business stakeholder, we've decided to just have a simple error occur if someone tries to withdraw
more than their current balance from the account. We'll iterate into more elaborate functionality in future iterations.
After having these conversations, using Gherkin syntax, we've defined the scope for the first iteration to cover the
following functionality.
```gherkin
Feature: Account Maintenance

Scenario: New account
Given I have a new account
 Then the account balance must be 0.00 USD

Scenario: Deposit money into account
Given I have an account with 0.00 USD
 When I deposit 5.00 USD
 Then the account balance must be 5.00 USD

Scenario: Withdraw money from account
Given I have an account with 11.00 USD
 When I withdraw 5.00 USD
 Then the account balance must be 6.00 USD

Scenario: Attempt to overdraw account
Given I have an account with 11.00 USD
 When I try to withdraw 50.00 USD
 Then the transaction should error
```
To add this to our project, I created a pkg/bankaccount folder in my project and placed the above syntax in a file
named account_behavior.feature in a folder named features under the bankaccount directory. Since [floats are not ideal for money](https://husobee.github.io/money/float/2016/09/23/never-use-floats-for-currency.html),
I've created money.go and money_test.go modeled after Google's [money protobuf](https://github.com/googleapis/googleapis/blob/master/google/type/money.proto)
and created regular unit tests for that type. Next, I'm created a simple account interface in account.go with functions to
return the balance, handle deposits, and handle withdrawals.  I'll create the implmentations for those after we've set up
our behavioral tests using the feature file we created. I next create the account_test.go file for my tests and since we'll
be focusing on testing behavior of the accounts there, I added it to the bankaccount_test package. So my account.go file
looks like this at this point.
```go
package bankaccount

type Account interface {
	Balance() Money
	Deposit(Money) error
	Withdraw(Money) error
}
```
and since we're going to be running Godog with Go test, we can start off adding a TestFeatures function to account_test.go.
```go
package bankaccount_test

import (
	"testing"

	"github.com/cucumber/godog"
)

func TestFeatures(t *testing.T) {
  suite := godog.TestSuite{
    ScenarioInitializer: func(s *godog.ScenarioContext) {
      // Add step definitions here.
    },
    Options: &godog.Options{
      Format:   "pretty",
      Paths:    []string{"features"},
      TestingT: t, // Testing instance that will run subtests.
    },
  }

  if suite.Run() != 0 {
    t.Fatal("non-zero status returned, failed to run feature tests")
  }
}
```
Note that the 'features' paths under the options matches the name of the features subdirectory where I placed my feature file.
I can run that test in my IDE (VS Code) at this point and see the following.
```shell

You can implement step definitions for undefined steps with these snippets:

func iDepositUSD(arg1, arg2 int) error {
	return godog.ErrPending
}

func iHaveANewAccount() error {
	return godog.ErrPending
}

func iHaveAnAccountWithUSD(arg1, arg2 int) error {
	return godog.ErrPending
}

func iTryToWithdrawUSD(arg1, arg2 int) error {
	return godog.ErrPending
}

func iWithdrawUSD(arg1, arg2 int) error {
	return godog.ErrPending
}

func theAccountBalanceMustBeUSD(arg1, arg2 int) error {
	return godog.ErrPending
}

func theTransactionShouldError() error {
	return godog.ErrPending
}

func InitializeScenario(ctx *godog.ScenarioContext) {
	ctx.Step(`^I deposit (\d+)\.(\d+) USD$`, iDepositUSD)
	ctx.Step(`^I have a new account$`, iHaveANewAccount)
	ctx.Step(`^I have an account with (\d+)\.(\d+) USD$`, iHaveAnAccountWithUSD)
	ctx.Step(`^I try to withdraw (\d+)\.(\d+) USD$`, iTryToWithdrawUSD)
	ctx.Step(`^I withdraw (\d+)\.(\d+) USD$`, iWithdrawUSD)
	ctx.Step(`^the account balance must be (\d+)\.(\d+) USD$`, theAccountBalanceMustBeUSD)
	ctx.Step(`^the transaction should error$`, theTransactionShouldError)
}

--- FAIL: TestFeatures (0.00s)
    --- FAIL: TestFeatures/New_account (0.00s)
        suite.go:449: step is undefined
    --- FAIL: TestFeatures/Deposit_money_into_account (0.00s)
        suite.go:449: step is undefined
    --- FAIL: TestFeatures/Withdraw_money_from_account (0.00s)
        suite.go:449: step is undefined
    --- FAIL: TestFeatures/Attempt_to_overdraw_account (0.00s)
        suite.go:449: step is undefined
FAIL
```
From this, I know that Godog is configured properly to find and parse my feature file. Since I don't have any implementations
for any of the steps at this point, it gives me some syntax I can use to start implementing my tests. Looking at my feature file,
I'm going to have some values created in one step which will be modified or read in subsequent steps. It may be tempting
to create some variables in account_test.go to store those values. But following good testing practices, each scenario
should be independent of each other and side effects or values of one scenario should not leak into other scenarios.
So using a context from the ScenarioInitializer in TestFeatures would be a better way to share values between steps
within the same test. Good tests follow an [arrange-act-assert pattern](https://automationpanda.com/2020/07/07/arrange-act-assert-a-pattern-for-writing-good-tests/),
so I'll organize my implemenation of my steps in my test file similarly.  Part of the reason for that is because they'll have similar
looking signatures.  Though there is nothing in gherkin or Godog that says a particular step must always be used with a
specific given, when, or then keyword, there are patterns for signatures of the step implementations that will lend them
to be most often used with a particular keyword. The arrange/given type steps will most often take a context and some step-specific
parameters and return a context, usually with some new value created as part of the setup of a test fixture. The act/when type
steps will often take a context and some step-specific parameters and either return a context after the behaviors being tested
have been invoked, and possibly an error if it is possible that the step might fail.  The assert/then type steps will take
a context and step-specific parameters, and return an error indicating whether the step has failed or not. These are not
hard/fast rules, just typical usage.

When adding or getting values from the context in our step implmentations, it's common that a struct will be used rather than
a string, to avoid any potential conflicts with other step implementations possibly using the same value for a string. So after
writing my step implementations and then the implementations of my account types, I wound up with the following test code.
```go
package bankaccount_test

import (
	"context"
	"fmt"
	"testing"

	"github.com/cucumber/godog"
	. "github.com/dumpsterfireproject/godog-examples/pkg/bankaccount"
)

// keys
type accountKey struct{}
type errorKey struct{}

//  helper methods
func getAccount(ctx context.Context) Account {
	acct := ctx.Value(accountKey{}).(Account)
	return acct
}

// Arrange steps
func iHaveANewAccount(ctx context.Context) context.Context {
	acct := NewSavingsAccount()
	return context.WithValue(ctx, accountKey{}, acct)
}

func iHaveAnAccountWith(ctx context.Context, units int, nanos int, currency string) context.Context {
	m, _ := NewMoney(currency, int64(units), int32(nanos))
	acct := NewSavingsAccount(WithBalance(m))
	return context.WithValue(ctx, accountKey{}, acct)
}

// Act steps
func iDeposit(ctx context.Context, units int, nanos int, currency string) (context.Context, error) {
	acct := getAccount(ctx)
	m, err := NewMoney(currency, int64(units), int32(nanos))
	if err != nil {
		return ctx, err
	}
	err = acct.Deposit(m)
	return context.WithValue(ctx, accountKey{}, acct), err
}

func iWithdraw(ctx context.Context, units int, nanos int, currency string) (context.Context, error) {
	acct := getAccount(ctx)
	m, err := NewMoney(currency, int64(units), int32(nanos))
	if err != nil {
		return ctx, err
	}
	err = acct.Withdraw(m)
	return context.WithValue(ctx, accountKey{}, acct), err
}

func iTryToWithdraw(ctx context.Context, units int, nanos int, currency string) context.Context {
	acct := getAccount(ctx)
	m, err := NewMoney(currency, int64(units), int32(nanos))
	if err != nil {
		return context.WithValue(ctx, errorKey{}, err)
	}
	err = acct.Withdraw(m)
	if err != nil {
		return context.WithValue(ctx, errorKey{}, err)
	}
	return context.WithValue(ctx, accountKey{}, acct)
}

// Assert steps
func theAccountBalanceIs(ctx context.Context, units int, nanos int, currency string) error {
	acct := getAccount(ctx)
	m, _ := NewMoney(currency, int64(units), int32(nanos))
	if !acct.Balance().IsEqual(m) {
		return fmt.Errorf("expected the account balance to be %s by found %s", m, acct.Balance())
	}
	return nil
}

func theTransactionShouldError(ctx context.Context) error {
	err := ctx.Value(errorKey{})
	if err == nil {
		return fmt.Errorf("the expected error was not found")
	}
	return nil
}

func TestFeatures(t *testing.T) {
	suite := godog.TestSuite{
		ScenarioInitializer: func(s *godog.ScenarioContext) {
			// Add step definitions here.
			s.Step(`^I have a new account$`, iHaveANewAccount)
			s.Step(`^I have an account with (\d+)\.(\d+) ([A-Z]{3})$`, iHaveAnAccountWith)
			s.Step(`^I deposit (\d+)\.(\d+) ([A-Z]{3})$`, iDeposit)
			s.Step(`^I withdraw (\d+)\.(\d+) ([A-Z]{3})$`, iWithdraw)
			s.Step(`^I try to withdraw (\d+)\.(\d+) ([A-Z]{3})$`, iTryToWithdraw)
			s.Step(`^the account balance must be (\d+)\.(\d+) ([A-Z]{3})$`, theAccountBalanceIs)
			s.Step(`^the transaction should error$`, theTransactionShouldError)
		},
		Options: &godog.Options{
			Format:   "pretty",
			Paths:    []string{"features"},
			TestingT: t, // Testing instance that will run subtests.
		},
	}

	if suite.Run() != 0 {
		t.Fatal("non-zero status returned, failed to run feature tests")
	}
}

```

And the following implemenation of my account type.

```go
package bankaccount

import (
	"fmt"
	"sync"
)

// Note that this is purely for example purposes and is not production code quality. I wrote my own
// implmentations here purely for the purpose of being able to demostrate some tests.

type Account interface {
	Balance() Money
	Deposit(Money) error
	Withdraw(Money) error
}

type SavingsAccount struct {
	balance Money
	sync.Mutex
}

type SavingsAccountOption func(*SavingsAccount)

func WithBalance(m Money) SavingsAccountOption {
	return func(s *SavingsAccount) {
		s.balance = m
	}
}

func NewSavingsAccount(opts ...SavingsAccountOption) *SavingsAccount {
	m, _ := NewMoney(USD, 0, 0)
	acct := &SavingsAccount{
		balance: m,
	}
	for _, opt := range opts {
		opt(acct)
	}
	return acct
}

func (s *SavingsAccount) Balance() Money {
	return s.balance
}

func (s *SavingsAccount) Deposit(m Money) error {
	s.Lock()
	newBalance, err := s.balance.Add(m)
	if err == nil {
		s.balance = newBalance
	}
	s.Unlock()
	return err
}

func (s *SavingsAccount) Withdraw(m Money) error {
	s.Lock()
	newBalance, err := s.balance.Subtract(m)
	if newBalance.IsNegative() {
		err = fmt.Errorf("withdrawal of %s would overdraw from balance of %s", m, s.balance)
	} else if err == nil {
		s.balance = newBalance
	}
	s.Unlock()
	return err
}
```

Now when I run my test, I see the following output.
```shell
Feature: Account Maintenance

  Scenario: New account                       # features/account_behavior.feature:3
    Given I have a new account                # account_test.go:23 -> github.com/dumpsterfireproject/godog-examples/pkg/bankaccount_test.iHaveANewAccount
    Then the account balance must be 0.00 USD # account_test.go:69 -> github.com/dumpsterfireproject/godog-examples/pkg/bankaccount_test.theAccountBalanceIs

  Scenario: Deposit money into account        # features/account_behavior.feature:7
    Given I have an account with 0.00 USD     # account_test.go:28 -> github.com/dumpsterfireproject/godog-examples/pkg/bankaccount_test.iHaveAnAccountWith
    When I deposit 5.00 USD                   # account_test.go:35 -> github.com/dumpsterfireproject/godog-examples/pkg/bankaccount_test.iDeposit
    Then the account balance must be 5.00 USD # account_test.go:69 -> github.com/dumpsterfireproject/godog-examples/pkg/bankaccount_test.theAccountBalanceIs

  Scenario: Withdraw money from account       # features/account_behavior.feature:12
    Given I have an account with 11.00 USD    # account_test.go:28 -> github.com/dumpsterfireproject/godog-examples/pkg/bankaccount_test.iHaveAnAccountWith
    When I withdraw 5.00 USD                  # account_test.go:45 -> github.com/dumpsterfireproject/godog-examples/pkg/bankaccount_test.iWithdraw
    Then the account balance must be 6.00 USD # account_test.go:69 -> github.com/dumpsterfireproject/godog-examples/pkg/bankaccount_test.theAccountBalanceIs

  Scenario: Attempt to overdraw account    # features/account_behavior.feature:17
    Given I have an account with 11.00 USD # account_test.go:28 -> github.com/dumpsterfireproject/godog-examples/pkg/bankaccount_test.iHaveAnAccountWith
    When I try to withdraw 50.00 USD       # account_test.go:55 -> github.com/dumpsterfireproject/godog-examples/pkg/bankaccount_test.iTryToWithdraw
    Then the transaction should error      # account_test.go:78 -> github.com/dumpsterfireproject/godog-examples/pkg/bankaccount_test.theTransactionShouldError

4 scenarios (4 passed)
11 steps (11 passed)
2.723471ms
PASS
```

In writing the tests first, it did give me some good cues on how to write the implemenation of the account types. For
example, when I looked at the iHaveANewAccount and iHaveAnAccountWith step implementations, it made it obvious that instead
of just having something like a NewSavingsAccount() function to create a new account, I'd want to start with an approach
like [functional options](https://dave.cheney.net/2014/10/17/functional-options-for-friendly-apis) to be able to create a
new account with a friendlier API that could handle a variety of cases where a new account might be created.

The code for this first post is available on [github](https://github.com/dumpsterfireproject/godog-examples/tree/post1).
This was just a simple example to get started showing how to add Godog to your project, how to get started with some
simple steps, how to share data between your steps, and how to run Godog using the Go testing library. By running Godog
with the Go testing library like this, this addresses some of the [limitations of previous releases](https://go-bdd.github.io/gobdd/) which
lead to the creation of other Gherkin implementations for Go. The recent efforts to 
[bring these libraries together](https://github.com/go-bdd/gobdd/issues/137) is a great thing for those in the Go community
looking to use Gherkin to do some BDD.

In my next posts, we'll move on to some of the other features that can be used to make your tests more powerful for
both you and the business users you're collaborating with to create your feature files.
