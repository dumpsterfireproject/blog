---
title: "ScalaTest – On Test Fixtures"
date: 2019-12-21
slug: "/scalatest-on-test-fixtures"
canonicalUrl: "https://thedumpsterfireproject.com"
description: TBD
tags:
  - Testing
  - Scala
---
A test fixture is something used to consistently test your code.  It’s the test data and/or resources used in your tests. It
may be some constant data, or it may be data or resources that require some set up prior to the test and clean up/tear down
after the test.  Each of your tests needs to run reliably whether it’s run as part of a suite or whether it’s run alone. That
means when sharing fixtures between tests to reduce redundant code and keep your test fixtures easy to maintain and update, the
state of your fixtures at the start of a given test can not be dependent on side effects of prior tests. So let’s look at a few
situations that may come up in maintaining your test fixtures and how to address them. All the following examples use the latest
3.1.0 release of ScalaTest.

# Constant Data

The simplest case is when your test fixture is a constant. It can be shared easily between tests being run in series or parallel.

```scala
package com.dumpsterfireproject.sharedvariable

import org.scalatest.flatspec.AnyFlatSpec
import org.scalatest.matchers.must.Matchers

class ExampleSpec extends AnyFlatSpec with Matchers {

  val subject: String = "Testing is"

  "Testing" must "be easy" in {
    s"$subject easy" mustBe "Testing is easy"
  }

  it must "be fun" in {
    s"$subject fun" mustBe "Testing is fun"
  }
}
```

If you have any test fixtures that have mutable state or require set up or clean up in between tests, this option quickly goes
out the window. As written above, if you switched subject to a var, the outcome of your test will be impacted by which other
tests passed or failed previously that may have mutated subject.

# Before/After

If I am using a var or mutable object to store some test fixture, I can ensure a test starts with a predictable state by using
a BeforeAndAfter or BeforeAndAfterEach trait. This will allow me to make sure the test fixtures are in a specific state before
each test is run and is cleaned up after each test is run.

```scala
package com.dumpsterfireproject.beforeandafter

import org.scalatest.BeforeAndAfterEach
import org.scalatest.flatspec.AnyFlatSpec
import org.scalatest.matchers.must.Matchers

class ExampleSpec extends AnyFlatSpec with Matchers with BeforeAndAfterEach {

  val buffer = new StringBuilder()

  override def beforeEach(): Unit = {
    buffer.append("Testing is ")
  }

  override def afterEach(): Unit = {
    buffer.clear()
  }

  "Testing" must "be easy" in {
    buffer.append("easy").toString mustBe "Testing is easy"
  }

  it must "be fun" in {
    buffer.append("fun").toString mustBe "Testing is fun"
  }
}
```

My opinion is that this overall reads fairly clearly. Personally though, I do find that as the number of text fixtures you are
using increases, it becomes more difficult to clearly communicate the intent and usage of all of them using this approach. A
more concrete drawback the test as written above, however, is that you now can’t run your tests in parallel predictably. You
want your tests to run quickly so that you run them often to get quick feedback as you’re coding. Long run times for tests
reduce the number of times you’ll run them locally. It also lengthens the run time of your test suite in any CI/CD pipeline
your using, which just draws out that feedback loop and diminishes the value you’re getting from that pipeline. Being able to
run your tests in parallel can help keep everything running quickly.

Additionally, if you have multiple test fixtures being used in your spec but not all the tests in your spec are using all the
test fixtures, you are still incurring the cost of setting up and cleaning up all your fixtures before and after all the tests,
regardless of whether they are used or not. When trying to keep the run time of your test suite to a minimum, repeated,
unnecessary set up/clean up can really have a negative impact.

One side note about choosing to use the BeforeAndAfter trait or the BeforeAndAfterEach, the ScalaTest docs have a good
explanation of each.

> Note: The advantage this trait has over BeforeAndAfterEach is that its syntax is more concise. The main disadvantage is that
> it is not stackable, whereas BeforeAndAfterEach is. I.e., you can write several traits that extend BeforeAndAfterEach and
> provide beforeEach methods that include a call to super.beforeEach, and mix them together in various combinations. By
> contrast, only one call to the before registration function is allowed in a suite or spec that mixes in BeforeAndAfter. In
> addition, BeforeAndAfterEach allows you to access the config map in its beforeEach and afterEach methods, whereas
> BeforeAndAfter gives you no access to the config map.
>
> [http://doc.scalatest.org/1.8/org/scalatest/BeforeAndAfter.html](http://doc.scalatest.org/1.8/org/scalatest/BeforeAndAfter.html)

# Get-Fixture Methods

One approach that can be used to help address some of the performance shortcomings of using BeforeAndAfter is to use some
get-fixture methods. Each test gets a new instance of a test fixture which will help enable you to parallelize your tests. You
only invoke the get-fixture method when you need it, eliminating unnecessary set up. You can have multiple get-fixture methods,
that may help add some expressiveness to your test code as well as minimize any set up overhead. The major drawback of this
approach is that there is no clean up of the fixtures after each test, if you do have clean up that needs to be performed.

```scala
package com.dumpsterfireproject.getfixture

import org.scalatest.flatspec.AnyFlatSpec
import org.scalatest.matchers.must.Matchers

class ExampleSpec extends AnyFlatSpec with Matchers {

  def fixture = new {
    val buffer = new StringBuilder(s"Testing is ")
  }

  "Testing" must "be easy" in {
    val f = fixture
    f.buffer.append("easy").toString mustBe "Testing is easy"
  }

  it must "be fun" in {
    val f = fixture
    f.buffer.append("fun").toString mustBe "Testing is fun"
  }
}
```

# Using Traits / Fixture-Context Objects

Another approach is to use fixture-context objects rather than get-fixture methods. You get the benefits of using get-fixture
methods with this approach. Additionally, it helps improve readability, as you don’t have to call a get-fixture method from
within your test. The code in your trait has already been executed when your test starts, and the body of your test method now
does not have any set up code in it.

```scala
package com.dumpsterfireproject.fixturecontextobject

import org.scalatest.flatspec.AnyFlatSpec
import org.scalatest.matchers.must.Matchers

class ExampleSpec extends AnyFlatSpec with Matchers {

  trait Buffer {
    val buffer = new StringBuilder("Testing is ")
  }

  "Testing" must "be easy" in new Buffer {
    buffer.append("easy").toString mustBe "Testing is easy"
  }

  it must "be fun" in new Buffer {
    buffer.append("fun").toString mustBe "Testing is fun"
  }
}
```

Additionally, you can stack traits to really make your test fixtures readable. You can either create a new trait that extends
another trait, or a test can be written something like `it must "pass" in new TraitA with TraitB`. The drawback of using
fixture-context objects is again the lack of clean up after each test.

# Overriding withFixture

Another approach to managing your test fixtures in ScalaTest is to override the withFixture method. Notice in the example below
rather than using AnyFlatSpec, we are now using FixtureAnyFlatSpec. There are two varieties of the withFixture method – one
that takes a NoArgTest as an argument, and another that takes a OneArgTest as an argument. The former is useful is you only
need to perform some side effect before executing a test, and the later will additionally allow you to reference your fixtures
from within your test. A NoArgTest or OneArgTest is a reference to the tests in your suite. You perform any set up of test
fixtures, call the super’s withFixture from within a try block, and then in the finally perform any cleanup. Assertion failures
are exceptions, so you’re not going to want to add a catch block that would not propagate those failures and make your test
appear to pass.

```scala
package com.dumpsterfireproject.withfixture

import org.scalatest.Outcome
import org.scalatest.flatspec.FixtureAnyFlatSpec
import org.scalatest.matchers.must.Matchers

class ExampleSpec extends FixtureAnyFlatSpec with Matchers {

  case class FixtureParam(buffer: StringBuilder)

  override def withFixture(test: OneArgTest): Outcome = {
    val buffer = new StringBuilder("Testing is ")
    try {
      withFixture(test.toNoArgTest(FixtureParam(buffer)))
    } finally {
      // clean up
    }
  }

  "Testing" must "be easy" in { fixture =>
    fixture.buffer.append("easy").toString mustBe "Testing is easy"
  }

  it must "be fun" in { fixture =>
    fixture.buffer.append("fun").toString mustBe "Testing is fun"
  }
}
```

The advantages of this approach are that it allows you to run your tests in parallel. Using case classes for your fixture data
can help make your fixture data more expressive and easy to read. A big advantage over using get-fixture methods or fixture
context objects is that you now had an option to perform any clean up after a test passes or fails. The potential drawback of
this approach is that the same set set up and clean up overhead is incurred for each test, which may result in longer run times
for your test if not all the tests in your spec actually need the fixtures being created.

Another advantage of this approach is that you can reference the attributes of your test from within the withFixture method.
Using dot notation, you have access to things like the name, tags, position, text, and the configMap of the current test.

# Loan-Fixture Methods

Another technique that can be used to manage your test fixtures is the loan fixture method. With this approach, you create a
function that executes your tests. The input to that function is a function that accepts your fixtures as input and produces
some output. That output and simply be Any to give you flexibility or it can be an Assertion if you want to enforce that your
tests end with an assertion. You can wrap the invocation of your test in a try, allowing you to also perform clean up after
your test has run. You can include different loan fixture methods if not all the tests in your spec use the same fixtures.
Overall, the approach provides a nice combination of flexibility and functionality.

```scala
package com.dumpsterfireproject.loanfixture

import org.scalatest.flatspec.AnyFlatSpec
import org.scalatest.matchers.must.Matchers

class ExampleSpec extends AnyFlatSpec with Matchers {

  def withBuffer(testCode: StringBuilder => Any) {
    val buffer = new StringBuilder("Testing is ")
    try {
      testCode(buffer)
    }
    finally {
      // clean up
    }
  }

  "Testing" must "be easy" in withBuffer { buffer =>
    buffer.append("easy").toString mustBe "Testing is easy"
  }

  it must "be fun" in withBuffer { buffer =>
    buffer.append("fun").toString mustBe "Testing is fun"
  }
}
```

# Before/After – Revisited

Note that when using the fixture methods discussed above, it anything fails in any set up or clean up, the exception is reported
as a test failure. That means that if one test fails, the other continue to run. Using a BeforeAndAfter or BeforeAndAfterEach,
if an exception occurs during set up or clean up, the entire suite is aborted and the failure reported. This may be useful in
short cutting the execution of a spec if some unrecoverable exception occurs during set up or clean up.

# Closing thoughts

All of these approaches have their use cases, their advantages and disadvantages. There is no one size fits all solution when
it comes to managing your test fixtures. Knowing each of these techniques and their strengths and weaknesses will help in
selecting the proper approach for each of your tests that will maintain a balance between maintainability and performance of 
your overall test suite.
