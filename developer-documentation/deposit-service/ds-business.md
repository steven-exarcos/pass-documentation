# Deposit Services - Business

Deposit Services system processes messages concurrently from both deposit and submission queues, utilizing dedicated 
listeners for each queue. This section outlines the workflow, detailing how messages are handled, processed, and how the
corresponding resources are managed and updated throughout the deposit lifecycle.

## Message flow and Business Services

Each message listener, one each for the `deposit` and `submission` queues, can process messages concurrently.

The `submission` queue is processed by the `SubmissionListener`, which resolves the `Submission` resource represented
in the message, and hands off processing to the `SubmissionProcessor`. The `SubmissionProcessor` builds
a `DepositSubmission`, which is the Deposit Services' analog of a `Submission` containing all the metadata and
custodial content associated with a  `Submission`. After building the `DepositSubmission`, the processor calls the 
`DepositTaskHelper` to perform the actual deposit. The `DepositTaskHelper` delegates the deposit steps to an instance
of a `DepositTask`. Importantly, the `SubmissionProcessor` updates the `Submission` resource in the repository as 
being _in progress_.

The `DepositTask` class contains the primary logic for packaging, streaming, and verifying the transfer of content from
the PASS repository to downstream repositories. The `DepositTask` will determine if the transfer of custodial content 
has succeeded, failed, or is indeterminable (i.e. an asynchronous deposit process that has not yet concluded). The 
status of the `Deposit` resource associated with the `Submission` will be updated accordingly.

## Failure Handling

A _failed_ `Deposit` or `Submission` is marked with `Deposit.DepositStatus = FAILED` or `Submission.AggregateDepositStatus = FAILED`.
When a resource has been marked `FAILED`, Deposit Services will ignore any messages relating to the resource. Failed `Deposit`
resources will be retried as part of the `DepositStatusUpdaterJob` job. Once all `Deposit` resource are successful, the
failed `Submission.AggregateDepositStatus` will be updated.

A resource will be considered as failed when errors occur during the processing of `Submission` and `Deposit` resources.
Some errors may be caused by transient network issues, or a server being rebooted. In the case of such failures,
Deposit Services will retry for n number of days after the `Submission` is created. The number of days
is set in an application property named `pass.status.update.window.days`.

`Submission` resources are failed when:

1. Failure to build the Deposit Services model for a Submission.
2. There are no files attached to the Submission.
3. Any file attached to the Submission is missing a location URI (the URI used to retrieve the bytes of the file).
4. An error occurs saving the state of the `Submission` in the repository (arguably a transient error).

For more details, refer to the `SubmissionProcessor`. Right now, when a `Submission` is failed, manual intervention may be required.
Deposit Services does retry the failed `Deposit` resources of the `Submission`. However, some of the failure scenarios
above must be resolved by the user. It is possible the end-user will need to re-create the submission in the user 
interface, and resubmit it.

`Deposit` resources are failed when:

1. An error occurs building a package.
2. An error occurs streaming a package to a `Repository` (potentially transient).
3. An error occurs polling (potentially transient) or parsing the status of a `Deposit`.
4. An error occurs saving the state of a `Deposit` in the repository (again, potentially transient).

See `DepositTask` for details. Deposits fail for transient reasons; a server being down, an interruption in network
communication, or invalid credentials for the downstream repository are just a few examples. As stated, DS will retry
failed `Deposit` resources for n number of days after the creation of the associated `Submission`. The number of days
is set in an application property named `pass.status.update.window.days`.

## Spring Error Handler

Certain Spring sub-systems like Spring MVC, or Spring Messaging, support the notion of a "global" `ErrorHandler`.
Deposit Services provides an implementation **`DepositServicesErrorHandler`**, and it is used to catch exceptions thrown
by the `DepositListener`, `SubmissionListener`, and is adapted as a `Thread.UncaughtExceptionHandler` and
as a `RejectedExecutionHandler`.

Deposit Services provides a `DepositServicesRuntimeException` (`DSRE`), which has a field `PassEntity resource`. 
If the `DepositServicesErrorHandler` catches a `DSRE` with a non-`null` resource, the error handler will test the type 
of the resource, mark it as failed, and save it in the repository.

In essence: `Deposit` and `Submission` resources will be marked as failed if a `DepositServicesRuntimeException` is 
thrown from one of the JMS processors, or from the `DepositTask`. As a developer, if an exceptional condition does 
**not** warrant a failure, then do not throw `DepositServicesRuntimeException`. Instead, consider logging a warning or 
throwing a `DSRE` with a `null` resource. Likewise, to fail a resource, all you need to do is throw a `DSRE` with a 
non-`null` resource. The `DepositServicesErrorHandler` will do the rest.

Since the state of a resource can be modified at any time by any actor in the PASS infrastructure, the 
`DepositServicesErrorHandler` encapsulates the act of saving the failed state of a resource within a `CRI`. The 
_pre-condition_ for updating the resource is that it must _not_ be in a _terminal_ state. For example, if the error 
handler is updating the state from `SUBMITTED` to `FAILED`, but another actor has modified the state of the resource to 
`REJECTED` in the interim, the _pre-condition_ will fail. Modifying the state of a resource after it reaches its 
_terminal_ state is not logical. In conclusion, the `DepositServicesErrorHandler` will not mark a resource as failed 
if it is in a _terminal_ state.

## Spring Boot Context

Deposit Services is implemented using Spring Boot, which heavily relies on Spring-based annotations and conventions to
create and populate a Spring `ApplicationContext`, arguably the most important object managed by the Spring runtime.
If you are unfamiliar with said annotations and conventions, the following resources would be beneficial to learning more
about them:

* [Baeldung](https://www.baeldung.com/spring-application-context)
* [Spring Boot Docs](https://docs.spring.io/spring-boot/documentation.html)

The entrypoint into the Deposit Services is the `DepositApp` class. Spring beans are created entirely in Java code by the 
`DepositConfig` and `JmsConfig` classes.

## Build and Deployment

Deposit Services' primary artifact is a single self-executing jar. In the PASS infrastructure, the Deposit Services
self-executing jar is deployed inside a simple Docker container.

Deposit Services can be built by running the following command:

```shell
mvn clean install
```

The main Deposit Services deployment artifact is located in `deposit-core/target/pass-deposit-service-exec.jar`. It is 
this jar file that is included in the [Docker image for Deposit Services](https://github.com/eclipse-pass/pass-support/pkgs/container/deposit-services-core), 
and posted on the [GitHub Release page](https://github.com/eclipse-pass/pass-support/releases).