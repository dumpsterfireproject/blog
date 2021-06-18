---
title: "Akka TestKit Quick Introduction"
date: 2020-02-07
slug: "/akka-testkit-quick-introduction"
canonicalUrl: "https://thedumpsterfireproject.com"
description: TBD
tags:
  - Testing
  - Scala
  - Akka
---
This is a quick introduction to setting up an asynchronous test of akka actor using TestKit and ScalaTest. With the Akka 2.6
release, Akka Typed becomes the default and the actors we've been using prior to the 2.6 release are now "Akka Classic." I'll be
covering Akka Typed in some forthcoming posts. But for now, I'll use Akka Classic to do a quick example of the basic usage of
TestKit. I'm going to demo this using a really simple actor that replies to a Ping message with a Pong.

```scala
package com.dumpsterfireproject.akka

import akka.actor.{Actor, Props}

object SampleActor {
  def props: Props = Props(new SampleActor)

  case object Ping
  case object Pong
  case object Troll
}

class SampleActor() extends Actor {
  override def receive: Receive = {
    case SampleActor.Ping => sender() ! SampleActor.Pong
    case SampleActor.Pong =>
      val origin = sender()
      origin ! SampleActor.Pong
      origin ! "expected Ping"
  }
}
```

For demonstration purposes later, if SampleActor is sent a Pong, it will reply with a Pong and a message that it expected a
Ping. Any other message will be ignored.

When doing asynchronous testing of actors, we can use TestKit and ScalaTest. We'll use a flat spec for this test. Since TestKit
is a class that we're extending, we can't also extend the AnyFlatSpec class like we've been doing in previous posts. We'll need
to extend the AnyFlatSpecLike trait to get the same functionality. We'll also want to extend the BeforeAndAfterAll trait, so we
can shut down the actor system when the test is complete. An example spec might look like the following.

```scala
package com.dumpsterfireproject.akka

import akka.actor.{ActorRef, ActorSystem}
import akka.testkit.{TestKit, TestProbe}
import org.scalatest.BeforeAndAfterAll
import org.scalatest.flatspec.AnyFlatSpecLike
import org.scalatest.matchers.must.Matchers

class ExampleSpec extends TestKit(ActorSystem("ExampleSpec"))
  with AnyFlatSpecLike with Matchers with BeforeAndAfterAll {

  override def afterAll(): Unit = TestKit.shutdownActorSystem(this.system)

  trait WithProbe {
    val probe: TestProbe = TestProbe()
    val sampleActor: ActorRef = system.actorOf(SampleActor.props)
  }

  "SampleActor" must "reply with Pong when sent a Ping" in new WithProbe {
    probe.send(sampleActor, SampleActor.Ping)
    probe.expectMsg(SampleActor.Pong)
  }

  it must "reply with a Pong when sent a Pong" in new WithProbe {
    probe.send(sampleActor, SampleActor.Pong)
    probe.expectMsg(SampleActor.Pong)
  }

  it must "not reply to Trolls" in new WithProbe {
    probe.send(sampleActor, SampleActor.Troll)
    probe.expectNoMessage()
  }
}
```

First, let's note the afterAll method. We need this to shut down the actor system so we can exit once the test is complete.
Next, I use fixture-context objects with the "WithProbe" trait. Other test fixture sharing techniques can other be used instead
if needed. For instance, you can use something like a loan fixture method instead if you needed to clean up some resources in a
finally block. The test probe is what you will use to send and receive messages to your actors under test. I create the actor in
this example in the WithProbe trait so that each test gets a new probe. I also create a new actor in the WithProbe trait to be
tested for each test. We don't want to have any messages sent or received in prior tests affecting the execution of subsequent
tests.

Let's first look at the "must reply with Pong when sent a Ping" test first. It's a pretty simple test. Using the probe, we sent
a message to the sampleActor. When the sampleActor receives the Ping, it replies to the sender with a Pong. The sender in this
case is the probe, so we use the expectMsg method to validate the reply sent by the sampleActor. If the response is not a Pong,
or no reply is sent, the test will fail. The expectMsg as written above will use the default timeout as defined in your
akka.test.single-expect-default setting. The expectMsg method is overloaded where you can pass in a specific timeout
FiniteDuration, or a specific timeout duration and a hint to help make your test output more readable in the event of a test
failure. There are plenty more methods available similar the the expectMsg method where you can use partial functions, or expect
specific types rather than specific objects, and more. I'll cover those in some upcoming posts, but for now we'll just stick
with expectMsg.

The "must reply with a Pong when sent a Pong" and "must not reply to Trolls" tests as written above have a potential problem,
which demonstrates why I used the fixture context object. The test as written is expecting a Pong when a Pong is sent. But
looking at the implementation of the SampleActor class, it also sends a string after the Pong. If we did not create a new probe
for each test and created one probe for the entire spec, the "must reply with a Pong when sent a Pong" test would pass when the
Pong is sent. But then the "must not reply to Trolls" would fail because the string that's sent after the Pong would still be
received by the probe, and the expectNoMessage would fail. The expectNoMessage as used above would use the default timeout from
the akka.test.expect-no-message-default setting. You can also pass in an explicit duration.

By using the WithProbe trait, since each test gets its own probe, both "must reply with a Pong when sent a Pong" and "must not
reply to Trolls" pass, since the side effects of one test executing does not bleed over to another test. Alternatively, I could
have just created the probe and actor in each test rather than in fixture sharing structure. It's nice to pull that code into a
reusable fixture rather than having duplicate code in each test.

One other thing worth mentioning here is that if we are in a situation where there may be messages that are sent that the test
doesn't need, but may prevent the test from passing, we can use TestProbe's ignoreMsg method. The ignoreMsg method takes a
partial function defining which messages can be ignored, and won't be considered when you do something like expectMsg. If we
wanted to ignore messages that were Strings, at some point before the probe sent the message to the sampleActor, we can do
something like the following.

```scala
probe.ignoreMsg {
  case s: String => true
  case _ => false
}
```

If we wanted to stop ignoring messages, you can then call the .ignoreNoMsg() method. Note that each time you call the .ignoreMsg
method, the partial function replaces any other partial function that we previously set. It is not cumulative.

One last note, rather than extending TestKit, you can extend ScalaTestWithActorTestKit. The later already extend
BeforeAndAfterAll and handles shutting down the actor system in the after all. If you override the afterAll in any of your
tests, don't forget to call super.afterAll to so that ScalaTestWithActorTestKit shuts down the actor system.
