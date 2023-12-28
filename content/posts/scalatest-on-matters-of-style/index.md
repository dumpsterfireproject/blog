---
title: "ScalaTest – On matters of style"
date: 2019-12-18
slug: "/scalatest-on-matters-of-style"
canonicalUrl: "https://thedumpsterfireproject.com"
description: TBD
tags:
  - Testing
  - Scala
---
> ### "Fashion changes, but style endures."
> ― Coco Chanel

ScalaTest provides great flexibility in styles of testing.  If you’re just getting started using ScalaTest and are migrating
from some other testing tools, ScalaTest provides some recommendations in the
["Selecting testing styles"](http://www.scalatest.org/user_guide/selecting_a_style) section of their [user guide](http://www.scalatest.org/user_guide).
But for those just getting started, which style should you choose?  ScalaTest recommends as a default choice FlatSpec
for unit and integration tests and FeatureSpec for acceptance tests. Those are pretty solid recommendations. But if you have a
curious mind, it’s hard not to poke around and explore the other styles.  It’s great to explore and try the other styles and
find which one really works best for your team. That might lead to the question, do we really need to settle upon certain styles
for certain levels of testing (unit, integration, acceptance)?  Are we stifling flexibility and creativity in favor of
conformity?

There was a great post recently from Google’s [Testing on the Toilet](https://testing.googleblog.com/2019/12/testing-on-toilet-tests-too-dry-make.html)
series about how to write tests.  It discusses the benefits
of reducing the amount of mental computation required for a reader to understand a test.  It emphasizes readability over
uniqueness.  Selecting a consistent style for the team helps keep the required mental computation to a minimum.  For example, if
a team adopts ScalaTest’s default recommendations, if I’m looking at a test and it’s using FeatureSpec, I would know straight
away it’s an acceptance test; if I were looking at a test written using FlatSpec, I would know straight away it was a unit or
integration test.  I would could understand some of the purpose of the test simply by recognizing the style.  When reading tests
written in a consistent style, your eyes and mind will become trained to quickly find test data and fixtures, assertions, set up
and tear down code, etc.  This allows you to spend more of your mental CPU time focusing on whether the test is correct rather
than just trying to navigate the test code.

This is similar to the principle of "Mise en place" that our designer discusses when explaining the user experience.  Wikipedia
decribes [Mise en place](https://en.wikipedia.org/wiki/Mise_en_place) as follows:

> Mise en place (French pronunciation:  [mi zɑ̃ ˈplas]) is a French culinary phrase which means "putting in place" or "everything
> in its place". It refers to the setup required before cooking, and is often used in professional kitchens to refer to
> organizing and arranging the ingredients (e.g., cuts of meat, relishes, sauces, par-cooked items, spices, freshly chopped
> vegetables, and other components) that a cook will require for the menu items that are expected to be prepared during a shift.

Choosing a consistent test style gives your test code some Mise en place. When you need to look for it, your assertions, test
fixtures, etc. are quickly found in the expected places and your focus remains on the correctness of the test rather than
spending time trying to find all the ingredients of the test.

Which style you choose is up to you and your team.  I appreciate the flexibility that ScalaTest provides.  After exploring the
various styles, I hope I have encouraged you to work with your team and choose consistent styles for your various test levels.
In case I haven’t quite convince you yet of the value of choosing consistent styles, I’ll leave you with this quote from
ScalaTest’s user guide: "In short, ScalaTest’s flexibility is not intended to enable individual developers to use different
testing styles on the same project. Rather, it is intended to enable project leaders to select a best-fit style or styles for
the team."