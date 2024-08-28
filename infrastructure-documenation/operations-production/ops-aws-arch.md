# Operations/Production - PASS Design & AWS Architecture

The PASS design supports flexible and scalable submission workflows of research publications and associated artifacts. 
It consists of four main components: PASS UI, Data Loaders, PASS Core, and Deposit Services. PASS Core acts as the 
central backbone, coordinating the workflow through, Elide, a JSON:API that interfaces with various external services. 
It stores documents using the Oxford Common File Layout (OCFL) on disk or through an S3 bucket, while the backend 
services are written in Java using Spring Boot. Data Loaders handle the ingestion of journal, grant, and publication 
data, while Deposit Services manage the deposits of submissions into various repositories like DSpace and NIH. The 
architecture is deployed primarily on AWS, leveraging services such as EC2, ECS, S3, and SQS to ensure high availability,
scalability, and security. This setup allows PASS to efficiently handle submission workflows and deposit services across
diverse environments, adhering to institutional policies and data governance standards. It is important to note that the
AWS architecture is a choice by JHU, but PASS can run on other cloud providers or run on infrastructure that is local to
the institution. The different components of PASS and their associated production setup and operations will be discussed.

A quick review of the PASS design and application architecture from our [Deployment and Architecture page](../../welcome-guide%2Fdeployment-architecture.md)
in the PASS Welcome Guide [welcome-guide](../../welcome-guide). Both the PASS design diagram and Application 
Architecture diagram are below for a quick reference:

<figure><img src="../../.gitbook/assets/pass-architecture-simple-v2-wo-admin-ui.png" alt="PASS Design Diagram"><figcaption><p>PASS Design Diagram</p></figcaption></figure>

<figure><img src="../../.gitbook/assets/application_architecture_diagram.jpg" alt="PASS Application Diagram"><figcaption><p>PASS Application Diagram</p></figcaption></figure>

## VPCs
The infrastructure operates across two VPCs (Virtual Private Clouds). The main VPC, hosts the PASS components and is 
equipped with both public and private subnets, without direct connectivity to the JHU network. The secondary VPC, 
includes a private subnet with connectivity to the JHU network, facilitating secure data access and integration.

## Application Load Balancers
There are two application load balancers (ALB), one for handling external traffic and another for internal communication 
between the different components of PASS. The public load balancers handles requests for the domain name. All traffic is
forwarded to HTTPS, and there is a security group that is attached, allowing only inbound/outbound requests to specific
ports. The other load balancer for internal communication has as similar setup but forwards to an internal host name.
Similarly, it has an attached security group for permitting specific ports for inbound/outbound requests.

## WAF
A web application firewall sits at the edge of the architecture boundary, as pictured in the PASS Application
Architecture diagram, between the end users and the PASS ALB. It is responsible for applying a set of rules to filter
out traffic and protect the internal virtual network.

## PASS Elastic Container Registry (ECR)


## Data Loaders

The Data Loaders (Journal, Grants, Publications) are executed as AWS Batch jobs within Fargate-based compute 
environments, each designed to accommodate specific data processing needs. The batch jobs leverage AWS services like S3
for configuration management, SQS for task scheduling and messaging, and ECS Fargate for executing the containerized 
data loaders. The architecture also integrates AWS IAM roles and policies to manage access controls, ensuring that data
loaders have the required permissions for accessing resources like S3 buckets and connecting to the PASS Core API.

## Submission UI (PASS UI)
PASS UI integrates with Shibboleth for authentication, providing a single sign-on experience for users. This ensures 
secure and streamlined access to the application, leveraging institutional identity providers for authentication.
PASS-UI interacts with AWS infrastructure by utilizing EC2 instances for hosting, ALBs for managing traffic, and S3 for
storing static assets.

## PASS Core
PASS Core is deployed on EC2 instances. These instances are part of the main VPC and are configured to run the PASS Core
API and associated services. Configuration files and deployment scripts for PASS Core and other components are stored in
S3, allowing for easy access and management during deployments and updates. In addition, the AWS Systems Manager 
parameter store contains configuration of environment variables in the PASS application. PASS Core uses SQS for message
queuing to handle asynchronous communication between different components of the PASS system. For example, when a 
submission is made, a message is placed in an SQS queue, which is then processed by the deposit services.

## File and Bytes Persistence
PASS Core uses Amazon S3 to store submission-related documents and metadata. The Oxford Common File Layout (OCFL) is 
employed to organize these files, providing options for local disk storage or S3 bucket retention. The configuration of
the File Service in PASS Core is done through environment variables. 

## Relational Database
PASS Core uses Amazon RDS, specifically a PostgreSQL database, to store and manage data related to publications, 
submissions, grants, and policies. The RDS instance resides within the same VPC and is not publicly accessible. The EC2
instances running PASS Core communicate with the RDS instance over the VPC’s internal network. Security groups are
configured to allow traffic between PASS Core and the RDS database, ensuring secure data access

## Deposit Services

## Publication Queue