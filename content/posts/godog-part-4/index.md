---
title: "Intro to Godog, Part 4: Backgrounds and Tags"
date: 2022-04-30
slug: "/godog-part-4"
description: Background and Tag Usage
tags:
  - Go
  - Testing
---
In part 2 of this series we introduced several hooks, including ones that could run before each scenario to handle set up
for each scenario or scenario outline example in the feature. That is a great place to do any setup work to make sure
that each scenario starts with a known state. But what if the business user that we're collaborating with has some 
needs to perform some work/setup before each scenario?  The business user really doesn't have visibility into the code
in the BeforeScenario hook. After all, we're using the .feature file in our collaboration with the business user, not
pair programming in Go. The background section of a feature would be a place where we can add some steps that execute
before each scenario and example of a scenario outline.

While we're discussing the background section with our business user, they also mention that it would be nice to add a
description to the scenario to provide more context to someone reading the feature file. In Gherkin, we can add free form
text under the feature line until we get to a line that starts with a keyword, like a background or scenario. This text
is ignored at runtime, but is available for reporting (we'll cover reporting in a later post). It can be plain text, or
it can be formatted text, such as mark-down. This can be a great place to include your acceptance criteria.
We can now add the following description and background to our feature.

```gherkin
Feature: Account Maintenance

As an account holder, I need to be able to depoit and withdraw money from my account,
and I expect the bank to be able to track the balance of my account accurately at all times.

Background: Setup new account
Given I have a new account

Scenario: New account
...
```

## Tags

Gherkin and Godog support adding tags to your test to be able to categorize and filter your tests. Tags can be placed
before a Feature, a Scenario, a Scenario Outline, or even an example table for a scenario outline. A tag cannot be associated
with a Background section, however. A tag starts with an `@` symbol, and can contain any non-white space characters.
Mutiple tags can be applied to a section of your feature, either by including mulitple tags on a single line like this:
```gherkin
@slow-running @issue#952
Scenario: Process ACH file
```
or on consecutive lines like this:
```gherkin
@slow-running
@issue#952
Scenario: Process ACH file
```
To include tags on a subset of examples in a scenario outline, you can include multiple example tables under a single
scneario outine and tag each table accordingly.
```gherkin
@concurrency
Scenario Outline: Opening accounts in different currencies
Given I have an account with 100.00 <currency>
 Then the account balance must convert to <dollars> USD

Examples:
|currency|dollars|
|CAD     |80     |
|CNY     |16     |
|EUR     |108    |

@wip
Examples:
|currency|dollars|
|CAD     |80     |
|CNY     |16     |
|EUR     |108    |
```

Tags can be very useful in filtering which tests are executed in a particular run.  E.g., maybe you want to skip your slow
running tests for now, skip tests marked work in progress, or run only tests associated with one particular business process.
I've even seen some people utilize the tags to generate some summary statistics in some customized reporting.
Some good examples of tags I've seen used successfully have included:
- A description of the functional area being tested, e.g., @deposits or @shipping
- A description of the type of test, e.g., @slow-running or @regression
- Indicating that a scenario is not yet ready to run; it is still a work in progress, e.g., @wip
- Associating tests with a particular issue or ticket, e.g.,@issue#952
- For custom reporting, utilizing a tag like @category(deposits) to summarize pass/failure rates by business process
- Categorizing tests by value, either to only run high priority tests, or as part of summary data in custom reports, e.g., @priority:high

### Tag Inheritance
When you have tags applied on some combination of features, scenarios, and examples, the tags are inherited from parent elements.
For example, say you had a feature tagged as @warehouse-management, a scenario outline under that feature tagged as
@receiving, and two example tables under the scenario outline, one tagged as @frozen and one tagged as @hazardous.
Each of those example tables would inherit the @receiving tag from the scenario outline and the @warehouse-management
tag from the feature.

### Applying filters to tags

At runtime, we can pass a tag argument to the go test command to filter which tests are executed. This can be useful if
we just want some fast feedback right now, and want to skip anything tagged as @slow-running, for example, and only run
those tests overnight. Or we might be looking for feedback on tests for a specific functional area, and only run tests
tagged for @shopping-cart. Or perhaps we just want to skip anything tagged as @flaky or @wip. The syntax for the tag filters
in Godog follow the [behat](http://behat.readthedocs.org/en/v2.5/guides/6.cli.html#gherkin-filters) syntax, where a tilde
is to negate/exclue a tag, && is used to 'and' two tags, and a comma is used to 'or' two tags.
A summary of syntax rules are:

| Example | Description |
| ----------- | ----------- |
| "@shipping" | run only scenarios with shipping tag |
| "~@wip" | exclude all scenarios with wip tag |
| "@shipping && ~@wip" | run shipping scenarios, but exclude wip |
| "@issue#129,@issue#137" | run scenarios tagged with issue#129 or issue#137 |

Tag filter expressions can include both and's and or's, where the and's are evaluated first and the or's are evalated
secondarily.

To demonstrate the use a tag filters, I have updated our account_test.go to accept tags from the command line. I moved the
"opt" variable outside of the TestFeatures function and then set the Tags value of the opt variable before creating the
test suite. The code is below.
```go
var tags = flag.String("godog.tags", "", "tags to execute")
var format = flag.String("godog.format", "pretty", "format")

var opts = &godog.Options{
	Paths: []string{"features"},
}

func TestFeatures(t *testing.T) {
	opts.TestingT = t
	opts.Tags = *tags
	opts.Format = *format

	suite := godog.TestSuite{
		ScenarioInitializer:  InitializeScenario,
		TestSuiteInitializer: IntializeTestSuite,
		Options:              opts,
	}

	if suite.Run() != 0 {
		t.Fatal("non-zero status returned, failed to run feature tests")
	}
}
```

Then running the tests after updating our feature to assign the @concurrency tag to the "Opening accounts in different currencies"
scenario outline and the @wip tag to the second example table, if I pass a tag filter of "@concurrency && @wip", only
that specific scenario outline will run, and for only the rows of the second example table.

```shell
$ go test pkg/bankaccount/account_test.go -v --godog.tags="@concurrency && @wip"
=== RUN   TestFeatures
Feature: Account Maintenance
  As an account holder, I need to be able to depoit and withdraw money from my account,
  and I expect the bank to be able to track the balance of my account accurately at all times.
=== RUN   TestFeatures/Opening_accounts_in_different_currencies

  Background: Setup new account
    Given I have a new account                               # account_test.go:27 -> *AccountTestState

    Examples:
      | currency | dollars |
      | CAD      | 80      |
=== RUN   TestFeatures/Opening_accounts_in_different_currencies#01
      | CNY      | 16      |
=== RUN   TestFeatures/Opening_accounts_in_different_currencies#02
      | EUR      | 108     |

3 scenarios (3 passed)
9 steps (9 passed)
1.562193ms
--- PASS: TestFeatures (0.00s)
    --- PASS: TestFeatures/Opening_accounts_in_different_currencies (0.00s)
    --- PASS: TestFeatures/Opening_accounts_in_different_currencies#01 (0.00s)
    --- PASS: TestFeatures/Opening_accounts_in_different_currencies#02 (0.00s)
PASS
ok  	command-line-arguments	0.337s
Brians-MacBook-Air:godog-examples brianhnat$
```

Additionally, you may want to consider adding the ability to pass in tag filters via environment tables instead of or
in addition to command line flags, especially if you make heavy use of environment variables within a CI/CD pipeline.

In future posts, we will still need to cover "rules" sections, new in Gherkin 6, and reporting. So keep an eye out for those.
