---
title: "OptionValues, EitherValues, and TryValues"
date: 2020-01-03
slug: "/optionvalues-eithervalues-and-tryvalues"
description: TBD
tags:
  - Testing
  - Scala
---
Another way to keep your ScalaTest specs concise and readable is to use the OptionValues, EitherValues, and TryValues traits
when testing Option, Either, or Try types. When testing options that should have a value, we could validate a value is defined
and then use get to validate the value.

```scala
"An option" must "have a value" in {
  val some: Option[Int] = Some(1)
  some mustBe defined
  some.get mustBe 1
}
```

Or by extending the org.scalatest.OptionValues trait, we can combine that into a single line.

```scala
"An option" must "have a value" in {
  val some: Option[Int] = Some(1)
  some.value mustBe 1
}
```

If option is a None, the .value would produce a TestFailedException with the message, "The Option on which value was invoked
was not defined." I like this syntax in that if you’re validating the values within an Option, you must expect it to be a Some.
Now your test focuses more on the expected contents rather then dealing with the Option. If you were expecting a None, you can
still simply use something like the following.

```scala
it must "have no value" in {
  val some: Option[Int] = None
  some mustBe empty
}
```

The org.scalatest.EitherValues trait works similarly.

```scala
"An either" must "be a right" in {
  val either: Either[String,String] = Right("hello")
  either.right.value mustBe "hello"
}

it must "be a left" in {
  val either: Either[String,String] = Left("hello")
  either.left.value mustBe "hello"
}
```

If you expected a left but found a right or vice versa, you would receive an error of "The Either on which left.value was
invoked was not defined as a Left" or "The Either on which right.value was invoked was not defined as a Right," respectively.

Likewise for the org.scalatest.TryValues trait.

```scala
"A Try" must "be a Success" in {
  val t: Try[String] = Success("A")
  t.success.value mustBe "A"
}

it must "be a Failure" in {
  val t: Try[String] = Failure(new Exception("oh, no!"))
  t.failure.exception.getMessage mustBe ("oh, no!")
}
```
