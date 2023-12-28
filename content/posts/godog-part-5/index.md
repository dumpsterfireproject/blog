---
title: "Intro to Godog, Part 5: Rules"
date: 2022-05-02
slug: "/godog-part-5"
canonicalUrl: "https://thedumpsterfireproject.com"
description: Intro to rules in Godog
tags:
  - Go
  - Testing
---

In this entry in my series on Godog I'm going to cover the new Rules keyword, introduced in Gherkin 6. The Rules keyword
is useful in example mapping. There is a great article explaining this new syntax at https://cucumber.io/blog/bdd/gherkin-rules/.

Instead of being restricted to Features that have Scenarios, you can do your example mapping such that your Feature has
one or more Rules, and those Rules are then illustrated by Examples (or Scenarios). So rather than writing this as we did
previously:

```gherkin
Feature: Account Maintenance

As an account holder, I need to be able to depoit and withdraw money from my account,
and I expect the bank to be able to track the balance of my account accurately at all times.

Scenario: Deposit money into a new account
  Given I have a new account
   When I deposit 5.00 USD
   Then the account balance must be 5.00 USD

Scenario: Deposit money into an existing account account
  Given I have an account with 100.00 USD
   When I deposit 50.00 USD
   Then the account balance must be 150.00 USD
```

We could write this in a format more like an example map:
```gherkin
Feature: Account Maintenance

As an account holder, I need to be able to depoit and withdraw money from my account,
and I expect the bank to be able to track the balance of my account accurately at all times.

Rule: I must be able to deposit into my own account

  Example: Deposit money into a new account
    Given I have a new account
     When I deposit 5.00 USD
     Then the account balance must be 5.00 USD

  Example: Deposit money into an existing account account
    Given I have an account with 100.00 USD
     When I deposit 50.00 USD
     Then the account balance must be 150.00 USD
```

Just like you can tag scenarios, you can also tag rules. And just like scenarios inherit any tags from features, your
rules will inherit any tags from features. Additionally, you can tag your Examples, and your Examples will also inherit
any tags from their Rules and Features.

Note also that you can use "Scenario" in place of "Example". In fact, looking at the gherkin-languages.json file in 
https://github.com/cucumber/gherkin-go, you can see easily which keywords can be used interchangeably. I'll illustrate this
using the entries for English, but the same holds for other languages defined in that file. The following keywords can
be used interchangeably:
- "Scenario" and "Example"
- "Scenario Outline" and "Scenario Template"
- "Examples" and "Scenarios"
- "Feature", "Business Need", and "Ability"

Also, in my last post, I covered how you can add some free form text as a description below your feature. I wanted to 
point out now, that the same holds true for Scenarios, Rules, and Examples. If you wanted to document, say, acceptance
criteria in your feature file for each example or for each rule or for each example, you have that ability. So in the
examples from above, they could look like this:

```gherkin
Feature: Account Maintenance

As an account holder, I need to be able to depoit and withdraw money from my account,
and I expect the bank to be able to track the balance of my account accurately at all times.

Scenario: Deposit money into a new account

As a new account holder.....

  Given I have a new account
   When I deposit 5.00 USD
   Then the account balance must be 5.00 USD

Scenario: Deposit money into an existing account account

As an existing account holder.....

  Given I have an account with 100.00 USD
   When I deposit 50.00 USD
   Then the account balance must be 150.00 USD
```

and:

```gherkin
Feature: Account Maintenance

As an account holder, I need to be able to depoit and withdraw money from my account,
and I expect the bank to be able to track the balance of my account accurately at all times.

Rule: I must be able to deposit into my own account

As an account holder making a deposit....

  Example: Deposit money into a new account

  As a new account holder.....

    Given I have a new account
     When I deposit 5.00 USD
     Then the account balance must be 5.00 USD

  Example: Deposit money into an existing account account

  As an existing account holder.....

    Given I have an account with 100.00 USD
     When I deposit 50.00 USD
     Then the account balance must be 150.00 USD
```

and Godog will parse those fine and make the descriptions available for reporting.

One quick note about using Rules. As I am writing this post, I am using Godog v0.12.5. There is an issue with the pretty 
formatter that already has a [pull request](https://github.com/cucumber/godog/pull/441) where Rules can only work with
the progress and junit formatters. If you do decide to start trying out Rules, be aware of this issue. I suspect it will
be completed and merged fairly soon, so don't let it discourage you from checking out Rules.
