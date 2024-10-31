# Notification Services - Business Logic

Notification Services listens for events during the submission process and generates email notifications to relevant 
users. The main components of the business logic are `SubmissionEventListener`, `NotificationService`, and `Composer`. 
The business logic handles events, composes notifications, and includes deep links that guide users directly to the 
necessary actions within PASS. It ensures that notifications are accurate, secure, and properly validated to maintain 
workflow integrity.

## SubmissionEventListener
The `SubmissionEventListener` is the component responsible for connecting to a messaging source and reading 
SubmissionEventMessage messages. Note the use of the Spring annotation `JmsListener` on the `processMessage` message. NS 
uses Spring for messaging configuration, which is managed by the `JmsConfig` class.

The NS responds asynchronously to modification of `SubmissionEvent` resources. A `SubmissionEvent` represents the state
of the `Submission` in a workflow. Each time a significant step in the submission workflow is taken, a `SubmissionEvent`
resource will be created. Notification Services do not create `SubmissionEvent` resources; `SubmissionEvent`
resources are created by Ember via Pass-Core as the user moves through the submission workflow. Pass-Core will emit 
a `SubmissionEventMessage` message indicating the creation of the `SubmissionEvent`, and the Notification Service will 
listen for, and respond to these messages.

## NotificationService
The `NotificationService` is the primary service that contains the business logic associated with reading the `Submission`
for the `SubmissionEventMessage` and composing a `Notification` and handing it off for dispatch. If future requirements 
dictate multiple `Notification`s were to arise from a single `SubmissionEvent`, `NotificationService` would be the 
starting point for implementing the distribution.

## Composer
The `Composer` class does the heavy lifting within the `NotificationService`. It is responsible for composing a `Notification`
from the `Submission` and `SubmissionEvent`, including:

* Determining the type of `Notification`.
* Creating and populating the data structures used in the parameters map.
* Invoking the LinkValidator to ensure the `SubmissionEvent` contains valid links for the `Notification` type.
* Determining the recipients of the `Notification`, and from whom the `Notification` should come from.

After a Notification has been created and populated, it is sent to the `DispatchService`.

### Notification Links

Notifications which prompt the recipient to perform an action are to contain a "deep link", that when clicked/activated, 
takes the user directly to the web page in PASS that requires their attention.  For example, a "please review" deep link 
would resolve directly to the submission overview, with a button allowing the user to approve the submission, cancel the 
submission, or request further changes. A notification that requests corrections should deep link to a page that 
highlights the edits or actions required by the user, displaying the comments or instructions from the reviewer.

#### Link Replay
Because deep links are durably stored (i.e. they can be found in email and used at any time after receipt), they may be 
replayed after the action has been taken. For example, a user activates a "please review" notification link after they 
have already performed the review. The user interface or link resolution components will respond in an appropriate way 
when this occurs.

#### Link Security
If a user's permissions change between the time a link was generated and the time a link was activated - for example, a 
User was an Authorized Submitter at the time the link was generated, and they were removed from the Grant by the time 
the link was activated, the user interface or link resolution components will respond in an appropriate way ensuring the 
user cannot approve something they don't have permission for.

#### User Token Link

According to the proxy submitter use case, a preparer may request an authorized submitter review a submission without 
that submitter existing within PASS. 

In this case, a `SubmissionEventMessage` with an associated `SubmissionEvent` of type`APPROVAL_REQUESTED_NEWUSER` will 
be created by Pass-Core and sent to the message queue. The `SubmissionEventMessage` object will have the User Token
Link set in the `SubmissionEventMessage.userApprovalLink` attribute. This is the link that is included in the
notification for the recipient to click on so a User object is created for them to authorize the submission.

#### Link Validation

The class `LinkValidator` will use `link-validators` in the notification configuration to validate that the correct URL 
is present in the `Link` object for the `Notification` type. The `Link` objects are created in the `SubmissionLinkAnalyzer` 
and then sent to `LinkValidator`. The `link-validators` field in the notification configuration file can be an array of
`Notification` types or `*`, which will be a link validation rule that applies to all.