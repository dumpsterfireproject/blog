---
title: "Testing the Future"
date: 2020-01-17
slug: "/testing-the-future"
description: TBD
tags:
  - Testing
  - Scala
---
ScalaTest's async test suites provide an easy and very readable way to write tests for your methods that return a Future. Using
Await.ready can really clutter your tests. ScalaTest's async test suites parallel the standard suites. For example, use
AsyncFlatSpec instead of AnyFlatSpec, AsyncFeatureSpec instead of AnyFeatureSpec, etc. I'll be using AsyncFlatSpec throughout
the rest of this post.

The first thing to know about the async test suites are that each test must end in either an Assertion or a Future[Assertion].
So you can mix tests of asynchronous and synchronous tests in the same spec. The compiler will enforce this. So in the following
example, the first two tests are valid, while the next two tests give compile errors. The last two tests are just for
illustrative purposes, first comparing what a similar test would look like using Await rather than utilizing the style of the
async test suits, and then using multiple assertions against the result of a future.

```scala
package com.dumpsterfireproject.async

import org.scalatest.matchers.must.Matchers
import org.scalatest.flatspec.AsyncFlatSpec
import scala.concurrent.Future

class ExampleSpec extends AsyncFlatSpec with Matchers {

  "Testing some synchronous code" must "return a Future[Assertion]" in {
    Future(7).map { i => i mustBe 7 }
  }

  "Testing some synchronous code" must "return an Assertion" in {
    1 mustBe 1
  }

  "Testing some synchronous code" must "does not compile in this case" in {
    Future(7) // produces type mismatch error
  }

  "Testing some synchronous code" must "does not compile in this case" in {
    1 mustBe 1
    println("hello, world") // produces type mismatch error
  }

  "Testing some asynchronous code" must "using Await for contrast" in {
    import scala.concurrent.Await
    import scala.concurrent.duration._
    Await.result(Future(7), 3.seconds) mustBe 7
  }

  "testing a string" must "pass" in {
    Future("Hello, World").map { s =>
      s must startWith("Hello")
      s must endWith("World")
    }
  }
}
```

Also know that an async test suite provides an execution context for your tests, so you don't need to import one for your tests
to run. Note that when your test returns a Future, it's the returned Future that determines whether your test will pass or fail
when it completes. So if I wrote the following test, my test would fail if either a == b or b == a returned false.

```scala
"Testing equality" must "be transitive" in {
  val a = 2
  val b = 2
  (a == b) mustBe true
  (b == a) mustBe true
}
```

But this next test passes as written, even though the first Future[Assertion] throws a TestFailedException. It's the
Future[Assertion] returned by the test that determines the pass or fail of the test. Even though at a quick glance it may
look like the previous example, there is no real validation of a == b in this test.

```scala
"Testing future equality" must "be transitive" in {
  val a = 2
  val b = 2
  Future(a == b).map( _ mustBe false ) // does not affect the pass/fail of the test
  Future(b == a).map( _ mustBe true )
}
```

If I wanted to include multiple Futures as above in my test, I could chain them together in a for-comprehension. The test
would fail when the first Future[Assertion] in the for-comprehension failed.

Validating failed Futures can be done using the recoverToSucceededIf and recoverToExceptionIf functions. The
recoverToSucceededIf function validates that a Future throws an exception and the type of exception thrown. It returns a
Future[Assertion]. If you want to validate any attributes of the exception, you can use recoverToExceptionIf. That will
return a Future[T] which is expected to fail. You can then map the failed Future to a Future[Assertion] as you validate any
attributes of the expected Exception. The following examples illustrate these behaviors.

```scala
"RecoverTo" must "validate that a future fails" in {
  recoverToSucceededIf[IllegalStateException] {
    Future.failed(new IllegalStateException)
  }
}

it must "fail when the wrong type of exception is thrown" in {
  // fails:
  //    Expected exception java.lang.IllegalStateException to be thrown, but java.lang.NullPointerException was thrown
  recoverToSucceededIf[IllegalStateException] {
    Future.failed(new NullPointerException)
  }
}

it must "fail when no exception is thrown" in {
  // fails:
  //    Expected exception java.lang.IllegalStateException to be thrown, but no exception was thrown
  recoverToSucceededIf[IllegalStateException] {
    Future(0)
  }
}

it must "validate how a future fails" in {
  recoverToExceptionIf[IllegalStateException] {
    Future.failed(new IllegalStateException("some message"))
  }.map { e => e.getMessage mustBe "some message" }
}
```

If you are using test fixtures in your tests, you can use FixtureAsyncFlatSpec. The withFixture method now uses a
complete/lastly where you can perform any cleanup, rather than a try/catch like you would use in the withFixture of a
FixtureAnyFlatSpec.

```scala
class ExampleFixtureSpec extends FixtureAsyncFlatSpec with Matchers {

  case class FixtureParam(buffer: StringBuilder)

  override def withFixture(test: OneArgAsyncTest): FutureOutcome = {
    val buffer = new StringBuilder("Testing is ")
    complete {
      withFixture(test.toNoArgAsyncTest(FixtureParam(buffer)))
    } lastly {
      // clean up
    }
  }

  "Testing" must "be easy" in { fixture =>
    Future(fixture.buffer.append("easy").toString).map(_ mustBe "Testing is easy")
  }
}
```

If you had cleanup to do conditionally depending on whether the test passed or fails, you can perform your cleanup using a
callback like onFailedThen, onSucceededThen, or onAbortedThen, among others.

```scala
override def withFixture(test: OneArgAsyncTest): FutureOutcome = {
  val buffer = new StringBuilder("Testing is ")
  withFixture(test.toNoArgAsyncTest(FixtureParam(buffer))) onFailedThen { t =>
      // clean up 
  }
}
```
