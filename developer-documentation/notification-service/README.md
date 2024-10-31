# Notification Services

Notification Services (NS) is a service component in PASS that provides timely notification (e.g. via email) of events
that occur as a Submission moves through a workflow. Notifications may be strictly informational ("a submission was
canceled", "a submission was approved"), or they may prompt for action ("please review and submit", "please correct
these things"). Notifications are directed to the actors participating in the submission process.

NS is a backend service component in PASS written in Java/Spring Boot. NS reacts asynchronously to `SubmissionEvent`
messages emitted by the Pass-Core component by composing and dispatching notifications in the form of emails to the
participants related to the event. NS is designed to be email-agnostic. For example, implementations could deliver 
notifications via Slack messages or interactively through reactive JavaScript. The only notification dispatch
**currently** implemented is email.

## Proxy Submission Use Case

This is currently the use case which Notification Services supports. Here are the high level use case steps:

1. A preparer user creates a Submission.
2. The authorized submitter user receives a notification that a Submission awaits their approval.
3. The authorized submitter user performs the submission, or requests changes from the preparer.
4. If the latter, the preparer updates the Submission, and the authorized submitter is notified. 
5. Supports edge cases, such as when the authorized submitter has never logged into PASS (i.e. the person who is going 
to click submit doesn't have a PASS User).

Note that NS is able to support many notification use cases, but the proxy submission notification is the only one 
currently implemented.

This guide steps through various topics on Notification Services:

* [Knowledge Needed / Skills Inventory](./ns-know-need.md)
* [Technologies Utilized](./ns-tech-util.md)
* Technical Deep Dive
  * [Model](./ns-model.md)
  * [Business Logic](./ns-business.md)
  * [Template](./ns-templates.md)
  * [Dispatch](./ns-dispatch.md)
  * [Configuration](./ns-configuration.md)
* [Institution Configuration](./ns-new-institution.md)
