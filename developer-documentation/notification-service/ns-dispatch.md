# Notification Services Dispatch

The Dispatch portion of Notification Services is not concerned with populating the Notification; it expects that 
business logic to have been performed earlier in the call stack.

Dispatch does have to adapt a Notification to the underlying transport used, in this case email. This means resolving 
User to recipient email addresses, and invoking the templating engine for composing email subject, body, and footer.

Each `SubmissionEventMessage` received by NS results in the creation of a single Notification, which triggers the 
dispatch of a single email. Multiple recipients (e.g. using CC or BCC email headers) can be specified on the email if 
needed. While email is the natural form of dispatching notifications, the model tries to remain independent of an 
underlying transport or dispatch mechanism.

## Email Implementation

The only `DispatchService` implementation is the `EmailDispatchImpl`, which is composed of three main classes:

* `Parameterizer`: responsible for resolving template content and invoking the templating engine, producing the content 
for the email subject, body, and footer.
* `EmailComposer`: responsible for adapting the Notification to an email (including resolving and setting the from, to, 
and cc addresses), provided the parameterized templates.
* `JavaMailSender`: responsible for actually sending the email to recipients.

## Composition

The `EmailComposer` is responsible for adapting the `Notification` to an email. This includes:

* Resolving Notification recipient URIs to email addresses.
* In the case of mailto URIs, the scheme specific part is used as the recipient.
* In the case of `User` object, the `User.email` is used.
* Applying the email recipient whitelist.
* Creating the email itself, including the email subject and message body, and encoding.
* The subject and message body are provided to the `EmailComposer` by the templating engine.

After the `EmailComposer` has created an email, it is returned to the `EmailDispatchImpl` for dispatch via SMTP.
