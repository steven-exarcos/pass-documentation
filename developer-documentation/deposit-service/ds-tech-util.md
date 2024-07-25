# Deposit Services - Technologies Utilized

As mentioned, DS is a backend service that is written in [Java](https://www.java.com/en/) and 
[Spring Boot](https://spring.io/projects/spring-boot).  The DS project is a [Maven](https://maven.apache.org/) 
project which is used to execute the standard lifecycle tasks for software development (i.e. build/test/package/release) 
of the DS service.  The Maven POM is a child of the `eclipse-pass/pass-support` POM which is a child of the `eclipse-pass/main` POM.

Spring Boot is used for functionality such as listening for JMS messages, configuration, “wiring-up” of DS components 
via dependency injection, and tests.

There are unit and integration tests in DS.  Tests are executed using [JUnit](https://junit.org/junit5/) and Spring Boot 
Test. [TestContainers](https://testcontainers.com/) are used for integration tests with pass-core.

[Docker](https://www.docker.com/) is used for building an DS docker image that is used for deployment.
