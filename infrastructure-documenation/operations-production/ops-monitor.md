# Operations/Production - Monitoring

## Alarms
There are several alarms setup in CloudWatch to monitor the health of the PASS system. Triggered and cleared alarms use
an SNS topic to send email to PASS devops. The following are a subset of Alarms monitor PASS:

* **Application Load Balancer (ALB)**
  * **HTTPCode_ELB_4XX_Count** - The number of HTTP 4XX client error codes that originate from the load balancer.
  * **TargetResponseTime** - The time elapsed, in seconds, after the request leaves the load balancer until the target
  starts to send the response headers.
  * **UnHealthyHostCount** - The number of unhealthy instances registered with your load balancer. An instance is 
  considered unhealthy after it exceeds the unhealthy threshold configured for health checks.
* **Simple Queue Service (SQS)**
  * **NumberOfMessagesSent** - The number of messages added to a queue.
  * **ApproximateAgeOfOldestMessage** - The approximate age of the oldest non-deleted message in the queue.
* **Simple Notification Service (SNS)**
  * **NumberOfNotificationsFailed** - The number of messages that Amazon SNS failed to deliver.
* **Elastic Compute Cloud (EC2)**
  * **CPUUtilization** - The percentage of physical CPU time that Amazon EC2 uses to run the EC2 instance, which 
  includes time spent to run both the user code and the Amazon EC2 code.
* **Elastic Cloud Service (ECS)**
  * **MemoryUtilized** - The memory being used by tasks in the resource that is specified by the dimension set that you're 
  using.
  * **CpuUtilized** - The CPU units used by tasks in the resource that is specified by the dimension set that you're 
  using.
  * **NetworkTxBytes** - The number of bytes transmitted by the resource that is specified by the dimensions that you're
  using. This metric is obtained from the Docker runtime.

There are a lot more metrics that can be collected and fine-tuned with different parameters. These are a sample of 
metrics that JHU PASS uses, but it is encouraged to see what metrics and their parameters that best fit the environment
PASS is in. More information regarding Cloud Watch and the metrics that can be monitored are on the [AWS documentation site](https://docs.aws.amazon.com/). 

## Synthetic Canary
This is a simple synthetic CloudWatch Canary script that is calling the application every 10 minutes. If 403(the login
page) is not returned, an alert will be triggered sending an email to PASS devops. Learn more about setting up a 
synthetic canary on the [Cloud Watch User Guide](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/CloudWatch_Synthetics_Canaries.html).

## Logs
CloudWatch Logs are used for logging from all PASS Docker containers. The Access Logs have been enabled on the Public 
ALB. These logs are written to an S3 bucket. In order to view/query the access logs, an AWS Athena database has been 
created. You can query the Athena database using standard SQL in the Query Editor.

## Dashboard
A dashboard has been added to CloudWatch that shows high level metrics for the PASS app, triggered alarms, and errors in
logs.