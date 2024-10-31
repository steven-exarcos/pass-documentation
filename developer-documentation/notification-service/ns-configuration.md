# Notification Services - Configuration

The required runtime configuration for Notification Services is composed of a configuration file and a set of 
environment variables.

## Configuration File
The NS configuration file is referenced by the environment variable `PASS_NOTIFICATION_CONFIGURATION` or a system property 
named `pass.notification.configuration`. The value of this environment variable must be a Spring Resource URI, beginning 
with classpath:/, file:/, or http://.

Here is a sample NS configuration file:

```json
{
  "mode": "${pass.notification.mode}",
  "recipient-config": [
    {
      "mode": "DEMO",
      "fromAddress": "demo-pass@mail.local.domain",
      "global_cc": [
        "demo@mail.local.domain"
      ],
      "whitelist": [
        "mailto:emetsger@mail.local.domain"
      ]
    }
  ],
  "templates": [
    {
      "notification": "SUBMISSION_APPROVAL_INVITE",
      "templates": {
        "SUBJECT": "Approval Invite Subject",
        "BODY": "Approval Invite Body",
        "FOOTER": "classpath:/templates/footer.hbr"
      }
    },
    {
      "notification": "SUBMISSION_APPROVAL_REQUESTED",
      "templates": {
        "SUBJECT": "Approval Requested Subject",
        "BODY": "Approval Requested Body",
        "FOOTER": "classpath:/templates/footer.hbr"
      }
    },
    {
      "notification": "SUBMISSION_CHANGES_REQUESTED",
      "templates": {
        "SUBJECT": "Changes Requested Subject",
        "BODY": "Changes Requested Body",
        "FOOTER": "classpath:/templates/footer.hbr"
      }
    },
    {
      "notification": "SUBMISSION_SUBMISSION_SUBMITTED",
      "templates": {
        "SUBJECT": "Submission Submitted Subject",
        "BODY": "Submission Submitted Body",
        "FOOTER": "classpath:/templates/footer.hbr"
      }
    },
    {
      "notification": "SUBMISSION_SUBMISSION_CANCELLED",
      "templates": {
        "SUBJECT": "Submission Cancelled Subject",
        "BODY": "Submission Cancelled Body",
        "FOOTER": "classpath:/templates/footer.hbr"
      }
    }
  ],
  "link-validators": [
    {
      "rels" : [
        "submission-view",
        "submission-review",
        "submission-review-invite"
      ],
      "requiredBaseURI" : "https://example.org",
      "throwExceptionWhenInvalid": true
    }, 
    {
      "rels": ["*"],
      "requiredBaseURI" : "http",
      "throwExceptionWhenInvalid": false
    }
  ]
}
```

## Mode

Notification Services has three runtime modes:
* `DISABLED`: No notifications will be composed or emitted.  All JMS messages received by NS will be immediately 
acknowledged and subsequently discarded.
* `DEMO`: Allows a whitelist, global carbon copy recipient list, and notification templates to be configured distinct 
from the `PRODUCTION` mode.  Otherwise, exactly the same as `PRODUCTION`.
* `PRODUCTION`: Allows a whitelist, global carbon copy recipient list, and notification templates to be configured 
distinct from the `DEMO` mode.  Otherwise, exactly the same as `DEMO`.

Configuration elements for both `PRODUCTION` and `DEMO` modes may reside in the same configuration file. There is no 
need to have separate configuration files for a "demo" and "production" instance of NS.

The environment variable `PASS_NOTIFICATION_MODE` (or its system property equivalent `pass.notification.mode`) is used 
to set the runtime mode.

## Notification Recipients

The recipient(s) of a notification (e.g. email) is a function of a `{Submission, SubmissionEvent}` tuple. After the 
recipient list has been determined, it can be manipulated as discussed below.

### Whitelist

Each configuration mode may have an associated whitelist. If the whitelist is empty, _all_ recipients for a given
notification will receive an email. If the whitelist is _not empty_, the recipients for a given notification will be
filtered, and _only_ whitelisted recipients will receive the notification. Having a whitelist for the `DEMO` mode is
useful to prevent sending test notifications to unintended recipients.

Production should use an empty whitelist (i.e. all potential notification recipients are whitelisted).

### Global Carbon Copy Support

Each configuration mode may specify one or more "global carbon copy" addresses. These addresses will receive a copy of 
each email sent by Notification Services (NS). Global carbon copy addresses are automatically whitelisted and do not 
need to be explicitly added to the whitelist. Blind carbon copy is also supported.

Here is an example recipient configuration that specifies a global carbon copy and a global blind carbon copy:

```json
{
  "recipient-config": [
    {
      "mode": "DEMO",
      "fromAddress": "pass-noreply@jhu.edu",
      "global_cc": [
        "pass-support@jhu.edu"
      ],
      "global_bcc": [
        "pass-ops@jhu.edu"
      ]
    }
  ]
}
```

Multiple email addresses may be specified.

### Example

For example, let's say that NS is preparing to send a notification to `user@example.org`. If the runtime mode of NS is `DEMO`
, and:
* The `DEMO` mode has no (or an empty) whitelist, then `user@example.org` and the global carbon copy address (for the `DEMO`
mode) receives the notification.
* The `DEMO` mode has a whitelist that does _not_ contain `user@example.org`, then only the global carbon copy address
receives the notification.
* The `DEMO` mode has a whitelist that _does contain_ `user@example.org`, then `user@example.org` and the global carbon
copy address receives the notification.
* The `DEMO` mode has a whitelist that does _not_ contain `user@example.org` and there is no global carbon copy address
(for the `DEMO` mode), then no notification will be dispatched.

If the runtime mode of NS is `PRODUCTION`, and:
* The `PRODUCTION` mode has no (or an empty) whitelist, then `user@example.org` and the global carbon copy address (for
the `PRODUCTION` mode) receives the notification.
* The `PRODUCTION` mode has a whitelist that does _not_ contain `user@example.org`, then only the global carbon copy
address receives the notification.
* The `PRODUCTION` mode has a whitelist that _does contain_ `user@example.org`, then `user@example.org` and the global
carbon copy address receives the notification.
* The `PRODUCTION` mode has a whitelist that does _not_ contain `user@example.org` and there is no global carbon copy
address (for the `PRODUCTION` mode), then no notification will be dispatched.

## Environment Variables

Supported environment variables (system property analogs) and default values are:

* `PASS_NOTIFICATION_QUEUE_EVENT_NAME` (`pass.notification.queue.event.name`): `event`
* `PASS_NOTIFICATION_MODE` (`pass.notification.mode`): `DEMO`
* `PASS_CORE_URL` (`pass.client.url`): `{PASS_CORE_URL}`
* `PASS_CORE_USER` (`pass.client.user`): `{PASS_CORE_USER}`
* `PASS_CORE_PASSWORD` (`pass.client.password`): `${PASS_CORE_PASSWORD}`
* `SPRING_MAIL_HOST` (`spring.mail.host`): `${SPRING_MAIL_HOST}`
* `SPRING_MAIL_PORT` (`spring.mail.port`): `${SPRING_MAIL_PORT}`
* `SPRING_MAIL_USERNAME` (`spring.mail.user`): `{SPRING_MAIL_USERNAME}`
* `SPRING_MAIL_PASSWORD` (`spring.mail.pass`): `{SPRING_MAIL_PASSWORD}`
* `SPRING_MAIL_PROTOCOL` (`spring.mail.transport`): `${SPRING_MAIL_PROTOCOL:SMTP}`
* `PASS_NOTIFICATION_CONFIGURATION` (`pass.notification.configuration`): `classpath:/notification.json`

In order for notification services to connect to AWS SQS queue (the default messaging provider), the following variables
must be set as Environment Variables (System Properties (-Dargs)):

* `AWS_REGION` (`aws.region`): AWS region id (i.e. `us-east-1`)

In order for NS to access to AWS SQS, standard AWS access management needs to be configured on the deployed NS. See 
[AWS IAM Access Management](https://docs.aws.amazon.com/IAM/latest/UserGuide/access.html) for more information.

For testing purposes, you may set AWS access keys to gain access:

* `AWS_ACCESS_KEY_ID` (`aws.accessKeyId`): AWS Access Key to account with access to SQS queue
* `AWS_SECRET_ACCESS_KEY` (`aws.secretKey`) : AWS Secret Access Key to account with access to SQS queue
  
**NOTE:** The AWS ID and key should only be used for testing, and in production access should be managed through IAM 
roles.
