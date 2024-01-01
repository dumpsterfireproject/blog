---
title: "Get a Clue"
date: 2019-12-28
slug: "/get-a-clue"
description: TBD
tags:
  - Testing
  - Scala
---
The withClue function in ScalaTest allows you to prepend an additional message to the test failure exception’s message. The
message is only output on a test failure.

```scala
package com.dumpsterfireproject.withClue

import org.scalatest.flatspec.AnyFlatSpec
import org.scalatest.matchers.must.Matchers

class ExampleSpec extends AnyFlatSpec with Matchers {

  "Increasing a person's age" must "be done in single year increments" in {
    withClue("The age of") {
      21 mustBe 20
    }
  }
}
```

Produces the following output.

```
The age of 21 was not equal to 20
 ScalaTestFailureLocation: com.dumpsterfireproject.withClue.ExampleSpec at (ExampleSpec.scala:10)
 Expected :20
 Actual   :The age of 21
```

This can be extremely useful in assertions and matchers that do not already provide a mechanism for providing a clue. For
example, assert does take an argument that provides a clue, while intercept does not. The withClue can be really useful when
reading logs in Jenkins, providing more information as to why a test failed even before you get chance to go back and look at
the test code. 
