---
title: "DRYer Tests"
date: 2020-01-24
slug: "/dryer-tests"
description: TBD
tags:
  - Testing
  - Scala
---
How you write your test code is subtly different than how you write your production code. Your tests should execute quickly, as
fast tests provide fast feedback. Yet the performance of your test code isn’t as critical as your production code. Your test
code should emphasize readability over uniqueness. It should minimize the amount of mental computation required for a reader to
understand a test. So the bar for "DRY" test code is lower than your production code.

Still your test code also need to be easily maintainable. It need to change easily as you continue to build your production
code. A great feature in ScalaTest to keep your test code readable and DRY is the table driven property check. When using table
driven property checks in your tests, you create a table with your examples. The first row of the Table are your table headings.
You iterate over your examples in Table using ScalaTest’s forAll function. You will get compiler errors if your Table structure
is not consistent. For example, if the number of arguments in your header row and the number of arguments in each of your
example rows are not all the same. You will also get compiler errors if the number and types of arguments in the function used
with your forAll do not your Table rows. Below is an example of using a table driven property check. Each row in this example
is a tuple of 2 Int’s, and my forAll is passed a function that has two Int’s as arguments.

```scala
package com.dumpsterfireproject.tableproperties

import org.scalatest.flatspec.AnyFlatSpec
import org.scalatest.matchers.must.Matchers
import org.scalatest.prop.TableDrivenPropertyChecks._

class ExampleSpec extends AnyFlatSpec with Matchers {

  val examples =
    Table(
      ("a",      "b"),
      (  1 + 2,   2 + 1),
      ( -1 + 2,   2 + (-1)),
      (  1 + 0,   0 + 1)
    )

  "addition" must "be transitive" in forAll(examples) { (a: Int, b: Int) =>
    a mustBe b
  }
}
```

You have flexibility as to where you use a forAll function in your test. Consider the following example of a feature spec. This
would give an error at runtime, "DuplicateTestNameException: Duplicate test name: Feature: transitivity Scenario: addition." I
could fix this by either moving the forAll inside of the scenario, so now I have one scenario named "addition" which iterates
over my examples. I could also use string interpolation in my scenario name, something like s"addition for $a and $b". Then
each scenario would have a unique name, and the name would describe the example being tested.

```scala
class FeatureSpec extends AnyFeatureSpec with Matchers {

  val examples =
    Table(
      ("a",      "b"),
      (  1 + 2,   2 + 1),
      ( -1 + 2,   2 + (-1)),
      (  1 + 0,   0 + 1)
    )

  Feature ("transitivity") {
    forAll(examples) { (a: Int, b: Int) =>
      Scenario ("addition") {
        a mustBe b
      }
    }
  }
}
```