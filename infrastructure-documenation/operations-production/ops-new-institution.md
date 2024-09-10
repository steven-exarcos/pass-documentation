# Operations/Production - Next Steps / Institution Configuration

Implementing and configuring PASS involves deploying the core application infrastructure using a variety of services 
provided by AWS. The institution must integrate with authentication systems, configure automated data loaders, set up 
backups, and monitor the application for performance and security. This guide provides a general outline of what must be
done to replicate a similar JHU infrastructure. 

* **Infrastructure Setup**
  * **VPC Setup:**
    * Create a VPC with both public and private subnets for deploying PASS. The public subnet can be used for the PASS 
    user interface, while private subnets can host the core API services and backend components like the database.
    * Ensure that security groups are defined to manage inbound and outbound traffic securely.
  * **EC2 and ECS for Core Services:**
      * Use EC2 instances to deploy PASS Core, PASS UI, and other related components.
  * **RDS for Database:**
    * Deploy the PostgreSQL database for PASS using Amazon RDS. This database will store submission data, policies, 
    grants, user information, and other metadata.
  * **S3 for Storage and Configuration Files:**
      * S3 is used for storing configuration files and temporary file storage submission artifacts.
  * **Load Balancers:**
    * Configure ALBs to handle incoming traffic for the PASS UI and ensure that traffic is routed to the correct backend
    services.
  * **SQS/SNS:**
    * SQS queues are used throughout the PASS application architecture. Notably they are used for messages related to 
    submissions, deposits, and submission events.
    * Configure notifications via SNS for alerts triggered by CloudWatch.
* **Security and Authentication**
  * **IAM Roles and Policies:**
    * Define and assign IAM roles to control which services can access different resources. For example, assign roles 
    for the ECS tasks to interact with S3, RDS, SQS, and other AWS services. When configuring the IAM role policies, the 
    recommendation is to follow best practices of [least privilege](https://aws.amazon.com/blogs/security/techniques-for-writing-least-privilege-iam-policies/).
  * **SSL and TLS**
    * Use AWS Certificate Manager (ACM) to manage SSL/TLS certificates for secure communication between users and the 
    PASS application. These certificates are attached to the load balancers.
  * **SSO Integration**
    * JHU is integrated with Shibboleth SSO, which enables JHU users to login with their institutional credentials. It 
    is possible to integrate with other SSO solutions for authentication, but may require some additional development to
    handle the header variables.
* **Data Loaders**
  * **Batch Jobs**
    * AWS Batch is composed of a Compute Environment, Job Queue, and Job Definition. AWS Batch uses the Job Definition 
    to run the PASS Data Loader job in the required VPC associated with the compute environment.
* **Scheduling**
  * **EventBridge**
    * Schedulers are used to run the Grant, Journal, and NIHMS Batch Jobs. The schedule will be dependent on the 
    institution, but the JHU implementation the Grant and NIHMS jobs run daily and the Journal job on a weekly basis.
* **Backup and Disaster Recovery**
  * **RDS Backups**
    * RDS has the capability to automate backups and provides a point-in-time recovery capabilities.
  * **S3 Versioning:**
    * Enabling version on S3 buckets will allow for data recovery in case of accidental deletions or overwrites.
* **Monitoring and Logging**
  * **CloudWatch**
    * Set up alarms and metrics to track CPU usage, memory utilization, and network traffic for various AWS resources.
    Some example alarms have been mentioned in the [Monitoring section](./ops-monitor.md#alarms). The exact alarms to be
    created are for the implementing institution.
  * **SNS**
    * By enabling notifications via SNS, alerts from CloudWatch can be sent to phone or email.