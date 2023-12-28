---
title: "Using Testcontainers-go In Your Integration Tests"
date: 2022-06-23
slug: "/testcontainers-go"
canonicalUrl: "https://thedumpsterfireproject.com"
description: Intro to using testcontainers in integration tests
tags:
  - Go
  - Testing
---

Here's a scenario I've come across multiple times when considering the database calls that your application makes
while you're writing your test code. In this example, let's assume we're creating some sort of API (rest, GraphQL, gRPC, or suc).
And let's assume we are persisting our data in some sort of database, maybe a relational database, a NoSQL data, or even a graph database.
When unit testing the code in your infrastructure layer that handles persistence of your data, mocking the database in those
unit tests makes sense. You have some tests that run quickly and give quick feedback when run. But how do we handle this as we
move further away from the infrastructure layer? Say we're writing unit tests against our service handlers behind the API. We
don't want to have to keep mocking database calls in code that is not responsible for persistence. Those tests would quickly
become overly complicated and brittle. The tests for the service handles could break whenever we made changes to the code handling persistence.
We'd break code we didn't touch. If we're in this situation, we have our abstractions and boundaries wrong and would want to
restructure our code so that we could unit test the service handlers without having to mock a database.

But even if our code was really clean and we had nice boundaries and abstractions, it would still be valuable to have some
integration tests to make sure the system behaved as expected when we had everything wired up from the API service handlers
to the data persistence. We'd want to run our unit tests first, to get the fast feedback from any failed unit tests. Once they
passed, we could create an instance of a database and seed some test data and run some integration tests to increase our
confidence that the system behaves as expected. Containers are a great way to handle this. Say the database is MySQL. You could
easily start up a MySQL container, run the DDL scripts to create the schema.  Around each integration test case, we could setup
and tear down some test data for that test. Once our tests are done, we could stop and remove the container. The whole process
could be easily repeatable running the tests multiple times on the same or different machines. Since creating and staring the container
is likely the most time-consuming piece of that whole process, we'd probably only want to do that once at the start of the test
suite rather than repeatedly before each test case.

In Go, TestMain is the place to handle setup and teardown like that before/after a test suite is executed. One way to manage
a database container in this approach is using the [Testcontainers-Go](https://golang.testcontainers.org) package. Below is
an example of a function that will start a MySQL container to be used in an integration test suite. We define the image to use
when creating the container. We configure the ports to be exposed that we will use to connect to the database. We configure
any environment variables, such as the database name or user/password. Those are all pretty standard when starting a container
from the command line. Note in the ContainerRequest there's also a WaitingFor parameter. When starting a MySQL container, the
container could go to running status before it is actually ready to start accepting connections. If we started running our integration
test cases at this point, they would fail, not being able to successfully connect to the database. So we configure a WaitingFor parameter,
in this case watching the container's log file for the text `port: 3306  MySQL Community Server - GPL` so that we know the container
is actually ready to accept connection requests. Once we've configured our ContainerRequest, we create the GenericContainer.
We can then get the host and port from the GenericContainer, create a valid connection string, and return a pointer to the container
and the connection string.

```go
type mysqlDBContainer struct {
    testcontainers.Container
    URI string
}

func setupMysqlContainer(ctx context.Context) (*mysqlDBContainer, error) {
    req := testcontainers.ContainerRequest{
        Image:        "mysql:8.0",
        ExposedPorts: []string{"3306/tcp", "33060/tcp"},
        Env: map[string]string{
            "MYSQL_ROOT_PASSWORD": dbPassword,
            "MYSQL_DATABASE":      dbName,
        },
        WaitingFor: wait.ForLog("port: 3306  MySQL Community Server - GPL"),
    }

    mysqlC, err := testcontainers.GenericContainer(ctx, testcontainers.GenericContainerRequest{
        ContainerRequest: req,
        Started:          true,
    })
    if err != nil {
        return nil, err
    }
    host, err := mysqlC.Host(ctx)
    if err != nil {
        return nil, err
    }
    port, err := mysqlC.MappedPort(ctx, "3306")
    if err != nil {
 		    return nil, err
    }
    url := fmt.Sprintf("%s:%s@tcp(%s:%s)/%s?parseTime=true", dbUsername, dbPassword, host, port.Port(), dbName)
    return &mysqlDBContainer{mysqlC, url}, nil
 }
```

Now in our TestMain, we can call this function to start the container. We can use a defer statement to handle the teardown of
the container after the test suite has completed. Typical usage in a TestMain function is to do something like this.

```go
func TestMain(m *testing.M) {
    exitCode := run(m)
    os.Exit(exitCode)
}
```

However, os.Exit will exit immediately and not call our teardown code, so we'll use a helper function to actually handle the setup
and teardown, as well as run the test suite. We'll call that helper function then from TestMain. One other thing to consider is 
that we might want to run our unit tests across all our packages. That should still run quickly. So we want to have a mechanism
to skip the setup and teardown of the container and the execution of the integration tests in that case. So we'll want the TestMain
that's running our integration test suite and our integration tests themselve to do something like check the -short flag in Go test.
So setting up our integration test suite might then look something like this.

```go
func TestMain(m *testing.M) {
    flag.Parse()
    if testing.Short() {
        exitCode:= m.Run()
        os.Exit(exitCode)
    }
    exitCode := run(m)
    os.Exit(exitCode)
}

func run(m *testing.M) int {
    ctx := context.Background()
    mysqlContainer, err := setupMysqlContainer(ctx, logger)
    if err != nil {
        // handle here
    }

    defer mysqlContainer.Container.Terminate(ctx)

    // perform any DB migrations/DDL scripts here
    // perform any other setup before all the tests run here, and defer any teardown

    return m.Run()
}
```

Then in the integration tests, they might generally look something like this.

```go
func TestFoo(t *testing.T) {
    if testing.Short() {
        t.SkipNow()
    }
    // the arrange section of your test
    setup() // call a setup function here, which will create the seed data required for this test
    defer teardown() // make sure you clean up your test data, so that side effects don't bleed over to other tests

    // add the act and assert sections of your test here
}
```

And you should be set to start implementing the details of your integration tests now!  Note that another approach that some projects
that I've worked on have taken involved creating a makefile which will handle the startup of your containers and execution of your tests.
The advantage of that we that when writing my code and running my tests frequently, I could just have a database container running the whole
time and not have to wait for the database container to start and stop each time I ran my tests. The downside was that the set up was a bit more
complicated now, bringing in tools like make, and the files were more spread out throughout the project. We did have multiple tasks in the makefile,
so having tasks to run the tests in the makefile did add some consistency. But also, I could easily start the database container and just
comment out the call the the Testcontainers setup and teardown while developing my code to avoid recreating the containers so many times, and then
just uncomment that code when ready to commit. So either using Testcontainers or using a makefile to handle starting the containers are
perfectly fine approaches. The choice really then comes down to what is a better fit for how you and your team work. I had been asked a question
recently about the Testcontainers approach, and thought it would be good time to add an example here.  Hope this was helpful!
