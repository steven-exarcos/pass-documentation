# Notification Services - Technologies Utilized

The code for NS can be found in the [pass-support repository](https://github.com/eclipse-pass/pass-support/tree/main/pass-notification-service)

NS is a backend service that is written in [Java](https://www.java.com/en/) and [Spring Boot](https://spring.io/projects/spring-boot).
The NS project is a [Maven](https://maven.apache.org/) project which is used to execute the standard lifecycle tasks for
software development (i.e. build/test/package/release) the NS service. The Maven POM is a child of the [`eclipse-pass/pass-support` POM](https://github.com/eclipse-pass/pass-support)
which is a child of the [`eclipse-pass/main` POM](https://github.com/eclipse-pass/main).

The following is a list of the most significant technologies used in NS:

* [Spring Boot](https://spring.io/projects/spring-boot) is used for functionality such as listening for JMS messages, email dispatch, configuration, 
“wiring-up” of NS components via dependency injection, and tests.

* There are unit and integration tests in NS. Tests are executed using [JUnit](https://junit.org/junit5/) and Spring Boot Test.
[TestContainers](https://testcontainers.com/) are used for integration tests with pass-core.

* [Docker](https://www.docker.com/) is used for building an NS docker image that is used for deployment.
