# Notification Services - Next Step / Institution Configuration

This section describes the configuration and setting up the Notification Services for a new institution.
The configuration process involves defining institution-specific settings, managing templates, and integrating with an
email system. Please note that the examples provided are for demonstration purposes and may require adjustments based on
your institution’s environment and external services setup.

## Prerequisites

* Email Service Configuration: Gather your institution's email service settings (e.g., SMTP host, port, authentication 
details) and ensure the email service is fully operational and ready for integration with Notification Services.
* Familiarity with Notification Services: Review the [Notification Services Knowledge Needed](./ns-know-need.md)
* AWS SQS Queue: Notification Services require an SQS queue for message handling. For testing purposes, you can use 
[LocalStack](https://www.localstack.cloud/) to simulate AWS services in a local environment. 

## Setup a Simple Configuration

This is a simplified version of the configuration. See the [configuration page](./ns-configuration.md) and
[template page](./ns-templates.md) for a more in-depth explanation of configuring NS. Be sure to replace email addresses
with those from your institution. Save this JSON file to a location that can be referenced by a Docker container or a 
running JAR file.

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

## Run Notification Services in Docker or JAR

The easiest way to get notification up and running is to get the latest NS image from our GitHub Container Registry.

Run the command below and be sure to replace `1.9.0` with the version you wish to run:

```shell
docker run --env=PASS_NOTIFICATION_CONFIGURATION=file:/ns-config/ns-config-json.json --env=AWS_REGION=us-east-1
--env=AWS_ACCESS_KEY_ID={YOUR_ID}
--env=AWS_SECRET_ACCESS_KEY={YOUR_KEY}
--volume=/path-to-ns-config-file
--workdir=/app 
--runtime=runc -d ghcr.io/eclipse-pass/pass-notification-service:1.9.0
```

The docker env variable `PASS_NOTIFICATION_CONFIGURATION` points to the volume mounted `/path-to-ns-config-file`, which
contains the JSON configuration. The `AWS_ACCESS_KEY_ID` and `AWS_SECRET_ACCESS_KEY` are your AWS access ID and key.
There are other environment variables to initialize that are found on the [configuration page](./ns-configuration.md#environment-variables)

**NOTE:** The AWS ID and key should only be used for testing and in production this should be managed through IAM roles.

You can also download the [Pass Support source code](https://github.com/eclipse-pass/pass-support/releases) and build the NS module to get the NS JAR file that can be 
invoked by running:

```shell
java -Dpass.notification.configuration=file:/ns-config/ns-config-json.json -Daws.region=us-east-1 -Daws.accessKeyId={YOUR_ID} -Daws.secretKey={YOUR_KEY} -jar pass-notification-service-exec.jar
```

The arguments `pass.notification.configuration`, `aws.region=us-east-1`, `aws.accessKeyId`, and `aws.secretKey` are the
minimum set to run the JAR; however, a fully functional NS will depend on the fulfillment of the [prerequisites](#prerequisites).

## Supporting Other Types of Queue Services

The default queue service is [AWS SQS](https://aws.amazon.com/sqs/). NS can be extended to support other types of queues,
but would require further development. Modifying the `JMSConfig` to support other JMS providers will enable support for 
another type of queue service.