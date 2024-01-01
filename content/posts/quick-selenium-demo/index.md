---
title: "Quick Selenium Demo"
date: 2020-02-21
slug: "/quick-selenium-demo"
description: TBD
tags:
  - Testing
  - Scala
  - Selenium
---
Using Selenium in Scala is pretty easy. For this demo, I am using a remote browser which I made available by using the
[standalone-chrome docker image](https://github.com/SeleniumHQ/docker-selenium). To start a container locally, you can
simply run the following.

```
docker run -d -p 4444:4444 -v /dev/shm:/dev/shm selenium/standalone-chrome
```

I’ll be using Selenium 3.141.59, which is the latest stable version at the time I wrote this post. Version 4.0.0 is in alpha,
and I will be sure to be checking that out really soon. One of the nice things about using a stand-alone chrome container is
that it is easy to incorporate that into your CI/CD pipeline without having to worry about having to have, say, Chrome installed
on a Jenkins node. There are also Firefox images available as well. Companies such as [SauceLabs](https://saucelabs.com/)
also provide remote browsers as a service. A very simple test that navigates to a web page and finds and element is as follows.

```scala
package com.dumpsterfireproject.selenium

import java.net.URL
import org.openqa.selenium.chrome.ChromeOptions
import org.openqa.selenium.remote.RemoteWebDriver
import java.util.concurrent.TimeUnit

object Selenium extends App {

  val remoteBrowserUrl = new URL("https://localhost:4444/wd/hub")
  val chromeOptions = new ChromeOptions
  chromeOptions.setHeadless(true)

  val driver = new RemoteWebDriver(remoteBrowserUrl, chromeOptions)
  driver.manage.window.maximize
  driver.manage.timeouts.implicitlyWait(1, TimeUnit.MINUTES)

  driver.get("https://example.com")
  val element = driver.findElementsByTagName("H1")
  println(element)
}
```

Since I’m using a standalone-chrome container, using the .setHeadless option is not necessary. I’ll get a headless browser
regardless, and could skip that. If you’re using a service to provide your remote browsers, you may have headed and headless
options available. You additionally could do something like start the chromedriver on a remote VM. If you are using the
chromedriver on your local machine, you can use ChromeDriver rather then RemoteWebDriver, and you won’t have to worry about
starting the chromedriver yourself; the ChromeDriver class will manage that for you.

One last note about the timeouts in Selenium. Often a one-and-done kind of check may not be reliable, so using a timeout,
whether explicit or implicit, so that the check can be performed multiple times until either the desired element is found or
the timeout expires will make for more reliable tests. Note that the waits are accomplished within the Fluent waits by
performing multiple checks with Thread.sleep in between. If you’re using Selenium in something like some [Akka](https://akka.io/docs/)
code, these Thread.sleep calls may not be the best thing to be running in your actor system’s thread pool, so you may want to
come up with an alternative approach to repeat checks within a timeout window.
