---
title: "Go Monorepos"
date: 2022-02-02
slug: "/go-monorepo"
description: Announcement of the migration to gatsby from wordpress
tags:
  - Go
  - git
---
I've been using Go for a year now. We started with developing a command line tool to use within a CI/CD pipeline
which handled user authentication and provide an easy way for a user to interact with one of our GraphQL APIs.
The experience using Go has been great. New team members sometimes find the learning curve with scala a bit steep
and the syntax unfamiliar. Looking for a way to reduce friction with ramping up new team members, I started exploring Go.
A year into it, I feel Go is an excellent choice for an easy to learn language that has powerful features when developing
SAAS applications. The ecosystem and community have been great.

Our team's baseline practices include using a monorepo. As we start to build out more micro-services in Go, I started
exploring if and how to use a monorepo with multiple Go services. I've found generally three approaches to monorepos in Go:
* Just don't
* Multiple modules in a single repo
* Multiple executables in a single module

## Options Reviewed

### Just Don't
The main argument for not doing a monorepo I've heard was that Go modules just weren't built to work that way. They
were designed for each module to be in its own repository.  A more interesting reason that I've heard is that a
monorepo can make semantic versioning confusing for team members. The tagging can get complicated. Tags will have to
be prefixed to avoid collisions in git tags. I can also be confusing for team members, in that it is natural for them
to look at the code in another module that is a dependency of their module to see how the code works. But if their code
is dependent on a semantic version from a tag that's not associated with the code they currently have checked out, it
can lead to some confusion. The visibility of code from dependency that is on a different branch or commit is an interesting
point. That would not be a draw back exclusive to Go modules. You could have that same issue if you're building jar files or npm packages
and publishing artifacts to an artifact repository rather than directly depending on the code in another module in your
monorepo.

### Multiple modules in a single repo
The next approach that I've seen taken is to have multiple Go modules in a single repository. Each module has its own go.mod and go.sum.
The layout is something like:
```text
- commandA
  - cmd
    main.go
  - pkg
  go.mod
  go.sum
- commandB
  - cmd
    main.go
  - pkg
  go.mod
  go.sum
- serviceA
  - cmd
    main.go
  - pkg
  go.mod
  go.sum
- serviceB
  - cmd
    main.go
  - pkg
  go.mod
  go.sum
```
As stated above, if using semantic versioning, it may be confusing having to look across multiple tags in your
repository. An approach taken to simplify that problem is to use `replace` statements in your go.mod files so that
the dependency is resolved using a relative path rather than semantic versioning.
```go
module mycompany.io/repo/client
go 1.17
require (
 mycompany.io/repo/core v0.0.0
)
replace mycompany.io/repo/core v0.0.0 => ../core
```

### Multiple executables in a single module
The last approach that I've seen taken is to just have a single module with multiple executables generated from that
module. The layout is something like:
```text
- cmd
  - commandA
    main.go
  - commandB
    main.go
- pkg
  - model
  - util
- services
  - serviceA
    main.go
  - serviceB
    main.go
go.mod
go.sum
```
The simplicity of this is really interesting. One criticism I might have is that in your go.mod file, it may not be
obvious which dependencies are used by which executables. It also seems the easiest of the approached to share code
in a way that starts to blur boundaries between various services or applications. I think this kind of approach can
be really useful, but also expects more discipline from your contibutors, in my opinion.

## Conclusion
As a small team that is already using a monorepo, sticking with a monorepo and using multiple modules in that repo makes sense.
We can utilize `replace` statements rather tagging for semantic versioning. While our number of services written in Go is small,
it won't yet be unmanageable to keep them in your head and make sure they have clear boundaries between modules. As
those services grow, our team may outgrow this approach. Using protobuf's at the boundaries of services will make it easier to evolve
interfaces without making breaking changes. There will be benefit in people being able to easily discover the actual
code in the dependencies. When we start to outgrow this approach, we can course correct. But for right now, I'll stick with
the mantra of [Yagni](https://www.martinfowler.com/bliki/Yagni.html).

## Background Articles
* [Golang Multimodule Monorepos](https://irilivibi.medium.com/golang-multimodule-monorepo-tutorial-3f5cf10e9b9a)
* [How to Golang Monorepo](https://medium.com/goc0de/how-to-golang-monorepo-4f62320a01fd)
* [Monorepo Golang application With Bazel](https://hardyantz.medium.com/getting-started-monorepo-golang-application-with-bazel-370ed1069b4f)
* [PROS AND CONS: GOLANG IN A MONOREPO](https://pliutau.com/pros_and_cons_golang_in_monorepo/)
* [Go Modules- A Guide for monorepos (Part 1)](https://engineering.grab.com/go-module-a-guide-for-monorepos-part-1)
* [Go in a Monorepo](https://blog.gopheracademy.com/advent-2015/go-in-a-monorepo/)
  * Note: This article predates go modules

### Example from Mobingi
* [Introducing ouchan, our main monorepo](https://tech.mobingi.com/2018/09/25/ouchan-monorepo.html)
* [Sample Repo](https://github.com/flowerinthenight/golang-monorepo)

Articles from Flowerinthenight's (Head of Engineering at Mobingi) personal blog:
* [Mobingi's golang monorepo](https://flowerinthenight.com/blog/2018/09/25/ouchan-monorepo)
* [A golang-based monorepo example](https://flowerinthenight.com/blog/2018/02/06/golang-monorepo)

### Other Presentations on YouTube and Examples
* [Uber Technology Day: Monorepo to Multirepo and Back Again](https://youtu.be/lV8-1S28ycM)
* [Untangling the Monorepo: Moving to Go Modules](https://youtu.be/OxxnE2Fi3a4)
* [Stellar Go Monorepo](https://github.com/stellar/go)