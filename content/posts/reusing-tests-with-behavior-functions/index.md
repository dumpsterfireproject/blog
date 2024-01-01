---
title: "Reusing Tests with Behavior Functions"
date: 2020-01-31
slug: "/reusing-tests-with-behavior-functions"
description: TBD
tags:
  - Testing
  - Scala
---
ScalaTest provides domain specific language to allow you to write behavior functions to share the same tests amongst different
fixture objects. What does this mean? If you are repeating the same tests in multiple specs, you can create some very readable
and compact syntax to encapsulate and reuse the tests. Create a function that encapsulates those tests. For most of the
TestSuites, you use "behave like" to call that function and execute all the tests. Let’s illustrate that with a quick example,
using a ridiculously oversimplified validation that a string is an valid email address.

```scala
package com.dumpsterfireproject.sharedtests

import org.scalatest.flatspec.AnyFlatSpec
import org.scalatest.matchers.must.Matchers

trait EmailValidationSpec { this: AnyFlatSpec with Matchers =>

  def emailValidation(validator: String => Boolean): Unit = {

    it should "return true when the string contains one at sign" in {
      validator("tester@example.com") mustBe true
    }

    it should "return false when the string does not contain an at sign" in {
      validator("tester") mustBe false
    }
  }
}

class ExampleSpec extends AnyFlatSpec with Matchers with EmailValidationSpec {

  val faultyValidator = (s: String) => true

  "The faulty email validator" must behave like emailValidation(faultyValidator)
}
```

Using the "behave like" syntax, I can invoke this same tests from any spec that properly extends the trait where the behavior
function is defined. As of ScalaTest 3.1, the "behave like" syntax is supported by free specs, flat specs, fun specs, and word
specs. If my test was a feature spec, my various scenarios would be provided in my behavior function. I would invoke them using
"ScenariosFor."

```scala
import org.scalatest.featurespec.AnyFeatureSpec

trait EmailValidationFeature { this: AnyFeatureSpec with Matchers =>

  def emailValidation(validator: String => Boolean): Unit = {

    Scenario("the string contains one at sign") {
      validator("tester@example.com") mustBe true
    }

    Scenario("the string does not contain an at sign") {
      validator("tester") mustBe false
    }
  }
}

class ExampleFeatureSpec extends AnyFeatureSpec with Matchers with EmailValidationFeature {

  val faultyValidator = (s: String) => true

  Feature("Email Validation") {
    ScenariosFor(emailValidation(faultyValidator))
  }
}
```
