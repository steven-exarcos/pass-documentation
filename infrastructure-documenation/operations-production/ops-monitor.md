# Operations/Production - Monitoring

## Alarms
There are several alarms setup in CloudWatch to monitor the health of the PASS system. Triggered and cleared alarms use
an SNS topic to send email to PASS devops. The following are a subset of Alarms monitor PASS:

* Application Load Balancer (ALB)
  * **HTTPCode_ELB_4XX_Count** - The number of HTTP 4XX client error codes that originate from the load balancer.
  * **TargetResponseTime** - The time elapsed, in seconds, after the request leaves the load balancer until the target 
  starts to send the response headers.
* Simple Queue Service (SQS)
  * **NumberOfMessagesSent** - The number of messages added to a queue.
* Simple Notification Service (SNS)
  * **NumberOfNotificationsFailed** - 

## Synthetic Canary
This is a simple synthetic CloudWatch Canary script that is calling the application every 10 minutes. If 403(the login
page) is not returned, an alert will be triggered sending an email to PASS devops.

## Logs
CloudWatch Logs are used for logging from all PASS Docker containers. The Access Logs have been enabled on the Public 
ALB. These logs are written to an S3 bucket. In order to view/query the access logs, an AWS Athena database has been 
created. You can query the Athena database using standard SQL in the Query Editor.

## Dashboard
A dashboard has been added to CloudWatch that shows high level metrics for the PASS app, triggered alarms, and errors in
logs.