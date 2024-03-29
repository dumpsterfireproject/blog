---
title: "Intro to Godog, Part 2: A Better Approach to Context & Using Hooks"
date: 2022-04-25
slug: "/godog-part-2"
description: Better handling of context between steps
tags:
  - Go
  - Testing
---
Picking up where we left off [in the last post](https://thedumpsterfireproject.com/godog-part-1), I wanted to put forth
another approach to managing the state between steps within a given scenario. We'll call it AccountTestState. In this approach, 
I'm going to create a struct to track any variables that I want to create or assign in one step and then use or validate in a 
subsequent step in the same scenario. I'm also going to create a function to reset that state, as we'll want to start each scenario 
with a clean state. To make things a little more readable, I'm going to pull the anonymous function assigned to the ScenarioInitializer 
property of the TestSuite created in the TestFeatures(t *testing.T) function into a separate function. There I'm going to allocate an instance
of an AccountTestState that will be used in each step.  I'm going to turn each of the functions I previously created into methods of the AccountTestState
type. By doing so, I'll be able to remove the Context from the arguments and return types of those step implementations. Since we want a clean
state at the beginning of each scenario, I'm going to introduce our first hook now. In the InitializeScenario I've created, I'll use a
BeforeScenarioHook to reset the AccountTestState at the start of each test. My test code now looks like the following.
```go
package bankaccount_test

import (
	"context"
	"fmt"
	"testing"

	"github.com/cucumber/godog"
	. "github.com/dumpsterfireproject/godog-examples/pkg/bankaccount"
)

type AccountTestState struct {
	account   Account
	lastError error
}

func (a *AccountTestState) reset() {
	a.account = nil
	a.lastError = nil
}

// Arrange steps
func (a *AccountTestState) iHaveANewAccount() {
	a.account = NewSavingsAccount()
}

func (a *AccountTestState) iHaveAnAccountWith(units int, nanos int, currency string) error {
	m, err := NewMoney(currency, int64(units), int32(nanos))
	a.account = NewSavingsAccount(WithBalance(m))
	return err
}

// Act steps
func (a *AccountTestState) iDeposit(units int, nanos int, currency string) error {
	m, err := NewMoney(currency, int64(units), int32(nanos))
	if err != nil {
		return err
	}
	err = a.account.Deposit(m)
	return err
}

func (a *AccountTestState) iWithdraw(units int, nanos int, currency string) error {
	m, err := NewMoney(currency, int64(units), int32(nanos))
	if err != nil {
		return err
	}
	err = a.account.Withdraw(m)
	return err
}

func (a *AccountTestState) iTryToWithdraw(units int, nanos int, currency string) error {
	m, err := NewMoney(currency, int64(units), int32(nanos))
	if err != nil {
		return err
	}
	// this step does not fail if the withdrawal fails; it just stores the error for later validation
	err = a.account.Withdraw(m)
	a.lastError = err
	return nil
}

// Assert steps
func (a *AccountTestState) theAccountBalanceIs(units int, nanos int, currency string) error {
	acct := a.account
	m, _ := NewMoney(currency, int64(units), int32(nanos))
	if !acct.Balance().IsEqual(m) {
		return fmt.Errorf("expected the account balance to be %s by found %s", m, acct.Balance())
	}
	return nil
}

func (a *AccountTestState) theTransactionShouldError() error {
	if a.lastError == nil {
		return fmt.Errorf("the expected error was not found")
	}
	return nil
}

func InitializeScenario(sc *godog.ScenarioContext) {
	ts := &AccountTestState{}
	sc.Before(func(ctx context.Context, sc *godog.Scenario) (context.Context, error) {
		ts.reset() // clean the state before every scenario
		return ctx, nil
	})
	// Add step definitions here.
	sc.Step(`^I have a new account$`, ts.iHaveANewAccount)
	sc.Step(`^I have an account with (\d+)\.(\d+) ([A-Z]{3})$`, ts.iHaveAnAccountWith)
	sc.Step(`^I deposit (\d+)\.(\d+) ([A-Z]{3})$`, ts.iDeposit)
	sc.Step(`^I withdraw (\d+)\.(\d+) ([A-Z]{3})$`, ts.iWithdraw)
	sc.Step(`^I try to withdraw (\d+)\.(\d+) ([A-Z]{3})$`, ts.iTryToWithdraw)
	sc.Step(`^the account balance must be (\d+)\.(\d+) ([A-Z]{3})$`, ts.theAccountBalanceIs)
	sc.Step(`^the transaction should error$`, ts.theTransactionShouldError)
}

func TestFeatures(t *testing.T) {
	suite := godog.TestSuite{
		ScenarioInitializer:  InitializeScenario,
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
When I run my test now, it passes the same as before. One advantage of taking this approach with the AccountTestState over the approach I used in the
last post where I simply added and read values from the Context is that it is now more explicit what variables are actually shared between my steps.

While we've introduced the BeforeScenarioHook to make sure we start each scenario with a clean state, let's take a quick look at some of the other hooks.
There is an AfterScenarioHook, which can be useful to clean up any side effects after your scenario has executed. This is a good time to maybe clean up
and files or data that may have been written by your test, or close and selenium browsers. Note that the AfterScenarioHook executes whether your scenario
passes or fails. The ScenarioContext also has StepContext, which has available a BeforeStepHook and AfterStepHook, where you can add any code to be executed
before and/or after each step. This can be useful in logging/reporting. Additionally, the AfterStepHook includes a StepResultStatus in its arguments, which
can be useful in conditionally doing any logging/reporting. For example, if your scenario involves some steps that use a Selenium driver, you could use an
AfterStepHook to capture any browser screen shots on failing Selenium steps, rather than having to include that log in every step implemenation that uses
the selenium driver.  Lastly, there are BeforeSuite and AfterSuite handlers that execute just one time each at the start and end of your test suite. These may
be useful in setting up any resources or data that are too expensive to be done before/after each scenario.  For example, you may want to start up and shut
down a database connection pool before/after your test suite's execution. Just be sure to not set up anything that has some shared that where the execution
of one scenario would have an effect that bleeds over into subsequent scenarios.

I've included below updates to my IntializeTestSuite function that leave placeholders to demonstrate where each of these hooks can be added.
```go
func InitializeScenario(sc *godog.ScenarioContext) {
	ts := &AccountTestState{}
	sc.Before(func(ctx context.Context, sc *godog.Scenario) (context.Context, error) {
		ts.reset() // clean the state before every scenario
		return ctx, nil
	})
	sc.After(func(ctx context.Context, sc *godog.Scenario, err error) (context.Context, error) {
		return ctx, nil
	})
	// Add step definitions here.
	sc.Step(`^I have a new account$`, ts.iHaveANewAccount)
	sc.Step(`^I have an account with (\d+)\.(\d+) ([A-Z]{3})$`, ts.iHaveAnAccountWith)
	sc.Step(`^I deposit (\d+)\.(\d+) ([A-Z]{3})$`, ts.iDeposit)
	sc.Step(`^I withdraw (\d+)\.(\d+) ([A-Z]{3})$`, ts.iWithdraw)
	sc.Step(`^I try to withdraw (\d+)\.(\d+) ([A-Z]{3})$`, ts.iTryToWithdraw)
	sc.Step(`^the account balance must be (\d+)\.(\d+) ([A-Z]{3})$`, ts.theAccountBalanceIs)
	sc.Step(`^the transaction should error$`, ts.theTransactionShouldError)
}

func IntializeTestSuite(sc *godog.TestSuiteContext) {
	sc.BeforeSuite(func() {
		// do any set up here one time before the entire test suite runs, e.g., create a database connection pool
		// that would be too expensive to do before each scenario.
	})
	sc.AfterSuite(func() {
		// do any clean up after the entire test suite is done executiong.
	})
	sc.ScenarioContext().StepContext().Before(func(ctx context.Context, st *godog.Step) (context.Context, error) {
		return ctx, nil
	})
	sc.ScenarioContext().StepContext().After(func(ctx context.Context, st *godog.Step, status godog.StepResultStatus, err error) (context.Context, error) {
		return ctx, nil
	})
}

func TestFeatures(t *testing.T) {
	suite := godog.TestSuite{
		ScenarioInitializer:  InitializeScenario,
		TestSuiteInitializer: IntializeTestSuite,
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

This post we stayed in the Go code that executes the feature file. In my next post, we'll move back to the feature file itself, where we can put the focus back
on some Gherkin syntax features and get back to collaborating with our business users.
