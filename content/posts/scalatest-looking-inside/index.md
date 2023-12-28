---
title: "ScalaTest – Looking Inside"
date: 2019-12-31
slug: "/scalatest-looking-inside"
canonicalUrl: "https://thedumpsterfireproject.com"
description: TBD
tags:
  - Testing
  - Scala
---
ScalaTest's Inside trait is a great way to keep your tests very expressive, clear, and readable. For the sake of example, let's
say we have the following traits and classes to test.

```scala
object SalesData {

  trait Address
  trait Order
  case class CustomerAddress(streetAddress: String,
                             city: String,
                             stateProvince: String,
                             postalCode: String,
                             country: String) extends Address
  case class Customer(firstName: String,
                      lastName: String)
  case class SalesOrder(customer: Customer,
                        shipToAddress: Address) extends Order
}
```

A blunt force approach to writing a test which validates an address is a CustomerAddress, and then validates data in the
address using only matchers might look as follows.

```scala
package com.dumpsterfireproject.inside

import org.scalatest.flatspec.AnyFlatSpec
import org.scalatest.matchers.must.Matchers
import SalesData._

class ExampleSpec extends AnyFlatSpec with Matchers {

  trait WithTestOrder {
    val address: Address = CustomerAddress("414 W 141st St", "New York", "NY", "95014", "US")
    val customer: Customer = Customer("Alexander", "Hamilton")
    val order: Order = SalesOrder(customer, address)
  }

  "An Address" must "have a valid state and postalCode" in new WithTestOrder {
    address mustBe a[CustomerAddress]
    address.asInstanceOf[CustomerAddress].stateProvince mustBe "NY"
    address.asInstanceOf[CustomerAddress].postalCode mustBe "10031"
  }
}
```

That's pretty verbose and the casting to CustomerAddress is a bit ugly. As we test the other attributes, the test code starts
to get pretty repetitive. The failure message is pretty basic.

```
"[95014]" was not equal to "[10031]"
```

We can eliminate the casting to CustomerAddress using have and arbitrary properties. This approach uses reflection to check
properties dynamically.

```scala
"An Address" must "have a valid state and postalCode" in new WithTestOrder {
  address mustBe a[CustomerAddress]
  address must have (
    'stateProvince ("NY"),
    'postalCode ("10031")
  )
}
```

That makes the test a bit easier to read. We also get a more descriptive error message.

```
The postalCode property had value "95014", instead of its expected value "10031", on object
CustomerAddress(414 W 141st St,New York,NY,95014,US)
```

But we lose autocomplete on the CustomerAddress attributes while we're writing the test. If I had mistakenly written my test
using ‘zipCode (“10031”) instead of postalCode, I would not catch my mistake until my test failed with the error, “have zipCode
(10031) used with an object that had no public field or method named zipCode or getZipCode.”

By using the org.scalatest.Inside trait, I can have a short, clear test that also provides autocompletion while writing the
test. So if ExampleSpec also extends Inside, I can implement my test as follows.

```scala
"An Address" must "have a valid state and postalCode" in new WithTestOrder {
  inside(address) { case CustomerAddress(_, _, stateProvince, postalCode, _) =>
    stateProvince mustBe "NY"
    postalCode mustBe "10031"
  }
}
```

The error message is very clear.

```
"[95014]" was not equal to "[10031]",
inside CustomerAddress(414 W 141st St,New York,NY,95014,US)
```

If the partial function did not have a match, the test would fail with an error message like the following. It's not the most
clear message, but the test does indeed fail.

```
The partial function passed as the second parameter to inside was not defined at the value passed as the first parameter to
inside, which was: TestAddress(414 W 141st St,New York,NY,95014,US)
```

You can also nest your checks, which does produce some nice output.

```scala
"A Sales Order" must "be shipped to a valid state and postalCode" in new WithTestOrder {
  inside(order) { case SalesOrder(customer, address) =>
    inside(address) { case CustomerAddress(_, _, state, postalCode, _) =>
      state mustBe "NY"
      postalCode mustBe "10031"
    }
  }
}
```

```
"[95014]" was not equal to "[10031]",
  inside CustomerAddress(414 W 141st St,New York,NY,95014,US),
inside SalesOrder(Customer(Alexander,Hamilton),CustomerAddress(414 W 141st St,New York,NY,95014,US))
```