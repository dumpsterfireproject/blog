---
title: "Basic Validation of Collections"
date: 2020-03-19
slug: "/basic-validation-of-collections"
description: TBD
tags:
  - Scala
  - Testing
---
Writing tests of collections in ScalaTest is easy. In this post, I'll cover a few of the very simplest cases. Of course the
simplest case is equality, something along the lines of validating two collections are equal.

```scala
"using equality" must "validate collections" in {
  val expected: Seq[Int] = Seq(1,2,3)
  Seq(1,2,3) mustBe expected
}
```

But the limitation of using equality if of course besides the same elements being in both the actual and expected, the number
and order of the elements must match. For instances where perhaps order doesn't matter or you only need to validate certain
elements, ScalaTest provides many other matchers for collections.

The first one we'll look at is "contains only." Only the elements in the right side of the matcher are allowed in the collection
being validated. Order and uniqueness of elements does not matter. The only matcher takes varargs as input, so if you're using
a collection as your expected value, you'll need to cast that collection appropriately.

```scala
"using only" must "validate collections" in {
  val expected: Seq[Int] = Seq(1,2,3)
  Seq(1,2,2,3) must contain only (1,2,3)
  Seq(1,2,2,3) must contain only (expected:_*)
}
```

The "allOf" matcher validates that the elements in the right side of the matcher are present in the collection being validated.
Order does not matter. The left side of the matcher is not restricted to only the elements of the right side. Where the
signature of "only" is `(Any*)`, the signature of "allOf" is `(first: Any, second: Any, remaining, Any*)`. So if your right side
is a collection itself, this one might not be the best fit.

```scala
"using allOf" must "validate collections" in {
  Seq(1,3,4,3,2) must contain allOf (1,2,3)
  
  val expected: Seq[Int] = Seq(1,2,3)
  expected match {
    case a :: b :: c => Seq(1,3,4,3,2) must contain allOf (a, b, c:_*)
  }
}
```

If your right side is a collection, uniqueness and order of the left side do not matter, and the left side is not restricted to
elements in the right side, "allElementsOf" may be a better option for you than "allOf". The "allElementsOf" matcher takes
GenTraversable[R] as its arguments.

```scala
"using allElementsOf" must "validate collections" in {
  val expected: Seq[Int] = Seq(1,2,3)
  Seq(1,2,2,3,4) must contain allElementsOf expected
}
```

The last matcher I'll cover in this post is "noneOf". It essentially provides a list of elements that are not allowed in the
collection being validated. Like "allElementsOf", the "noElementsOf" matcher takes GenTraversable[R] as its arguments.

```scala
"using noElementsOf" must "validate collections" in {
  val expected: Seq[Int] = Seq(1,2,3)
  Seq(4,5,6) must contain noElementsOf expected
}
```

There are many more matchers available to validate collections using ScalaTest that I may cover in future posts. As usual, the
ScalaTest user guide and scala docs are excellent sources of information. Learning the behavior of each matcher, paying
attention to the signatures of each matcher, will enable to to write some clean validations of your collections.
