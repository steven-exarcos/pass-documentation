# Notification Services - Steps for supporting new institution

This section describes the configuration and setting up the Notification Services for a new institution.
The configuration process involves defining institution-specific settings, managing templates, and integrating with an
email system.

## Prerequisites

* Gather your email service's settings (e.g., SMTP settings) and ensure the Email service is ready for integration.
* Read through the [Notification Services Knowledge Needed](./ns-know-need.md)
* Understand how to run [PASS Docker](../../welcome-guide/setup-run-pass.md) and pull individual docker images.

## Run Notification Services in Docker or Jar

The easiest way to get notification up and running is to get the latest NS image from our GitHub Container Registry. Run
the command below and be sure to replace `1.9.0-SNAPSHOT` with the version you wish to run:

```shell
docker pull ghcr.io/eclipse-pass/pass-notification-service:1.9.0
```

You can also download the Pass Support source code and build NS module to get the NS jar file that can be invoked by 
running:

```

```

## Simple Configuration

This configuration is a simplified version of the configuration. See the [configuration page](./ns-configuration.md) and
[template page](./ns-templates.md) for a more in-depth explanation of configuring NS . Ensure to replace email addresses
with the email address from your institution.

```json
{
  "recipient-config": [
    {
      "mode": "DEMO",
      "fromAddress": "no-reply@institution.edu",
      "global_cc": ["support@institution.edu"],
      "global_bcc": ["admin@institution.edu"]
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
    }
  ]
}
```
