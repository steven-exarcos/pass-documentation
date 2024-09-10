# Operations/Production - Next Steps / Institution Configuration

Implementing and configuring PASS involves deploying the core application infrastructure using a variety of services 
provided by AWS. The institution must integrate with authentication systems, configure automated data loaders, set up 
backups, and monitor the application for performance and security. This guide provides a general outline of what must be
done to replicate a similar JHU infrastructure. 

* **Infrastructure Setup (AWS Deployment)**
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
      * S3 is used for storing configuration files, temporary file storage, and
  * **Load Balancers:**
    * Configure ALBs to handle incoming traffic for the PASS UI and ensure that traffic is routed to the correct backend
    services.
  * **SQS/SNS:**
* **Security and Authentication**
  * **IAM Roles and Policies:**
  * **SSL and TLS**
  * **SSO Integration**
* **Data Loaders**
  * **AWS Batch Jobs**
  * **S3 and Configuration Files**
* **Backup and Disaster Recovery**
  * **RDS Backups**
  * **S3 Versioning:**
* **Monitoring and Logging**
  * **CloudWatch**