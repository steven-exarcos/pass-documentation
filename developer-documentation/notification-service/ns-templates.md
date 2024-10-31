# Notification Services - Templates

Templates are used to customize the subject, body, and footer of email messages that result from a notification. Each 
notification type has a corresponding template, and the templates and their content are configured in the 
`notification.json` configuration file. A sample portion of the configuration is below:

```json
{
  "templates": [
    {
      "notification": "SUBMISSION_APPROVAL_INVITE",
      "templates": {
        "SUBJECT": "Approval Invite Subject",
        "BODY": "Approval Invite Body",
        "FOOTER": "Approval Invite Footer"
      }
    },
    {
      "notification": "SUBMISSION_APPROVAL_REQUESTED",
      "templates": {
        "SUBJECT": "Approval Requested Subject",
        "BODY": "Approval Requested Body",
        "FOOTER": "Approval Requested Footer"
      }
    },
    {
      "notification": "SUBMISSION_CHANGES_REQUESTED",
      "templates": {
        "SUBJECT": "Changes Requested Subject",
        "BODY": "Changes Requested Body",
        "FOOTER": "Changes Requested Footer"
      }
    },
    {
      "notification": "SUBMISSION_SUBMISSION_SUBMITTED",
      "templates": {
        "SUBJECT": "Submission Submitted Subject",
        "BODY": "Submission Submitted Body",
        "FOOTER": "Submission Submitted Footer"
      }
    },
    {
      "notification": "SUBMISSION_SUBMISSION_CANCELLED",
      "templates": {
        "SUBJECT": "Submission Cancelled Subject",
        "BODY": "Submission Cancelled Body",
        "FOOTER": "Submission Cancelled Footer"
      }
    }
  ]
}
```

You can see that there is an object identifying each notification type, and for each notification type a `SUBJECT`, 
`BODY`, and `FOOTER` template may be defined.

The value associated with `SUBJECT`, `BODY`, and `FOOTER` may be inline content as seen in the example, or it can be a 
reference to a Spring Resource URI, e.g.:

```json
    {
      "notification": "SUBMISSION_APPROVAL_INVITE",
      "templates": {
        "SUBJECT": "classpath:/templates/submission-approval-subject.txt",
        "BODY": "classpath:/templates/submission-approval-body.txt",
        "FOOTER": "classpath:/templates/submission-approval-footer.txt"
      }
    }
```

Using Spring Resource URIs to refer to the template location is a more flexible and maintainable way of managing 
notification templates. It allows the templates to be updated in place without having to edit the primary configuration
file (`notification.json`) any time a template needs updating. Using Spring Resource URIs also allows template content
to be shared across notification types. For example, each notification type could use the same`FOOTER` content. The
`CompositeResolver` is responsible for determining whether the value represents inline content, or if it represents a
Spring Resource URI to be resolved.

Notification Services supports Mustache templates, specifically implemented using Handlebars. Each template is injected 
with the `parameters` map from the `Notification`. See above for the documented fields of the `parameters` map. It is 
beyond the scope of this README to provide guidance on using Mustache or Handlebars, but there are some examples in 
`pass-docker`, and in the `HandlebarsParameterizerTest`. Both inline template content and referenced template content 
(i.e. Spring Resource URIs) can be Mustache templates.
