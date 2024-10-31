# Operations/Production - How to Deploy

In this article the JHU instance of PASS will be used to demonstrate how we deploy the application to production. As 
with the JHU architecture decisions, deployment will be done in various ways and will likely vary between institutions 
depending on their architecture. It is important to note that there is a process of testing and stepping through 
development and staging environments before deploying to production. This articles will focus on one deployment workflow
to a production environment.

## GitHub (GH) Automations

[GitHub Actions](https://docs.github.com/en/actions) is used to deploy PASS to our production environment. We keep the 
GH Workflow YAML and Python scripts in a private repository since this is a JHU specific deployment. The main workflow 
`Production Deployment` is run from the GH web interface and releases a specified version of the main branch
from the PASS repositories. The only parameter required to run the workflow is the release version. The 
`Production Deployment` workflow is responsible for initiating the entire deployment process which encapsulates other 
workflows. The following GH workflows are part of the deployment:

1. `Production Deployment:`
   * This is the main production deployment workflow, which orchestrates the deployment of various components of an 
   application to the production environment. It is triggered manually with a specified release version. The workflow 
   consists of multiple jobs that utilize two workflows (`Publish to SNS Topic: Triggers Deployment to AWS` 
   and `Deploy PASS Services and Data Loaders to ECS AWS`) to deploy different parts of the application, such as PASS 
   Core, Deposit Services, Notification Services, and the Data Loaders. Each job specifies its environment as 
   `Production` and uses the release version as the Docker image tag. The release version is set by clicking the
   `Run workflow`button and specifying the `Release Version`.

2. `Publish to SNS Topic: Triggers Deployment to AWS:`
   * This workflow is designed to deploy an application to AWS by triggering an AWS SNS (Simple Notification Service) 
   topic. It accepts inputs such as environment, commit reference, secrets, and Docker image tag. The workflow includes 
   steps to checkout the repository, set up Python, install necessary Python packages (such as boto3 for AWS 
   interactions), and run a Python script that publishes a message to the SNS topic, initiating the deployment process 
   in AWS.

3. `Deploy PASS Services and Data Loaders to ECS AWS:`
   * This workflow focuses on deploying specific services and data loaders to AWS ECS (Elastic Container Service). 
   Similar to the previous workflow, it takes inputs like environment, Docker image name, secrets, and Docker image tag.
   The steps involve checking out the repository, setting up Python, installing necessary packages, and running a Python
   deployment script that deploys the specified services and data loaders to AWS.

### Steps Performed During the Deployment to Production

1. **Trigger the Deployment**:
   * The deployment to production is initiated manually through the `workflow_dispatch` event, requiring a specified 
   release version.
2. **Deploy PASS Core/UI Application**:
   * The `deploy_pass_app` job uses the `Publish to SNS Topic: Triggers Deployment to AWS` workflow to deploy the core 
   application to the production environment. It provides the necessary inputs, including environment (`Production`), 
   commit reference, and Docker image tag, all set to the specified release version.
3. **Deploy Deposit Services**:
   * The `deploy_deposit_services` job uses the `Deploy PASS Services and Data Loaders to ECS AWS` workflow to deploy the deposit
   services to AWS ECS. It specifies the Docker image name (`deposit-services-core`) and tag as the release version.
4. **Deploy Notification Services**:
   * The `deploy_notification_services` job also uses the `Deploy PASS Services and Data Loaders to ECS AWS` workflow to
   deploy notification services. It sets the Docker image name to `pass-notification-service` and the tag to the release 
   version.
5. **Deploy Grant Loader**:
   * The `deploy_grant_loader` job deploys the grant loader using the same workflow, with `jhu-grant-loader` as the 
   Docker image name.
6. **Deploy Journal Loader**:
   * The `deploy_journal_loader` job deploys the journal loader, specifying `pass-journal-loader` as the Docker image 
   name.
7. **Deploy NIHMS Loader**:
   * Finally, the `deploy_nihms_loader` job deploys the NIHMS loader, using `pass-nihms-loader` as the Docker image 
   name.

Each job inherits secrets necessary for AWS access and uses specific Docker images corresponding to different components
of the application, all tagged with the release version to ensure consistency across the deployment. The Docker images
final destination is the Elastic Container Registry (ECR) in the [PASS application architecture](./ops-aws-arch.md#pass-elastic-container-registry-ecr).