---
title: "Testing Your Code"
date: 2019-12-15
slug: "/testing-your-code"
description: TBD
tags:
  - Testing
---
I have a confession to make.  I have not been the most diligent at writing tests for my code throughout my career.  Over this
past year our team has committed to measuring our test coverage and have set quarterly goals for improvements in test coverage.
The experience of putting such a deliberate focus on testing has been an extremely positive one. Perhaps what's been motivating
about this experience is some sort of gamification effect.  Maybe I'm just the Pavlovian dog and the code coverage number going
up is my whistle.  Maybe it's the ["what you measure is what you get"](https://www.isixsigma.com/community/blogs/what-you-measure-what-you-get/)
effect. 

But much more than the actual numbers, what has been really motivating about this experience is how it has improved the actual
code that I write.  Those improvements in the actual code that is written are what will continue to provide the motivation to
keep the focus in writing tests even after specific goals and have been achieved and new and different goals have been set.

It encourages me to write code in a more functional style.  The majority of the code that I write is in scala, with some
javascript sprinkled in.  There are many benefits stated for functional programming, but the ones in particular that I find in
regard to this topic are that pure functions are easier to reason about and easier to test.  Eliminating side effects and state
preconditions/mutations makes writing tests easier as well.  It reduces the hidden inputs and free variables.  Alvin Alexander
explained hidden inputs and free variables as ["Grandma's Cookies,"](https://alvinalexander.com/scala/fp-book/benefits-of-functional-programming)
the secret ingredients the make your code work correctly.

It also encourages reducing the cyclomatic complexity of your functions and your code in general.  [Cyclomatic complexity](https://www.guru99.com/cyclomatic-complexity.html)
is the measure of independent paths through your code.  That is, the more possible branches you have through a function, the
more work it's going to take to fully test that function.  Reducing the cyclomatic complexity of the code that I write has
resulted in function that are smaller, easier to read, reason about and reuse, and easier to test.

Writing smaller functions that have good coverage can also save you a lot of time when there is a bug in your code. You can
usually zero in on where the bug exists much more quickly when each function is concise and specific.  Comparing the existing
test cases against the use case of the reported bug helps navigate quickly to where the bug exists.

It has also really improved the way we use dependency injection.  Dave Gurnell does a really great job of discussing and
comparing some of the various dependency injection approaches and tools in a [Scala Central Presentation](https://www.youtube.com/watch?v=OJe0Dm3t5wQ).
Even if you're not programming in scala, the constructor based approach can be used very broadly in many languages and doesn't
require any additional tools.  It will encourage you to program to interfaces and makes it really easy to create mock
implementations to use during testing.

For my testing, I most commonly use ScalaTest.  My next few entries will be focused on writing tests with [ScalaTest](http://www.scalatest.org/).
They will cover readability, managing test data, and selecting test styles. 
