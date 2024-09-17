# Eclipse PASS Deployment Guide

## Table of Contents
1. [Summary](#summary)
2. [Deployment Architecture](#deployment-architecture)
3. [AWS Infrastructure Components](#aws-infrastructure-components)
4. [PASS Deployment Process](#pass-deployment-process)
5. [PASS Release Process](#pass-release-process)
6. [Related Information](#related-infomormation)

## Summary

The Public Access Submission System (PASS) is an open-source platform designed to streamline compliance with funder and institutional open access policies. This guide outlines the deployment process for PASS, which is adaptable to various architectures including cloud, hybrid, or on-premises environments.

> **Note**: PASS is transitioning towards a cloud-native version. Expect ongoing changes to the architecture, infrastructure, and deployment process, such as moving from Docker Compose to Kubernetes or implementing Infrastructure as Code with Terraform. See (roadmap)[./roadmap.md] for more information.

## AWS Infrastructure Components

The current PASS infrastructure in AWS includes:

- **EC2**: Hosts Docker Compose
- **ECS**: Hosts auxiliary microservices
- **RDS**: Stores metadata
- **S3**: Stores binary data (managed by OCFL)
- **ALB**: Provides SSL for the frontend
- **WAF**: Protects the frontend

## Deep Dive: Deployment & Release 

### Prerequisites
- Docker and Docker Compose
- Git

### PASS Deployment Process

1. Install dependencies:
   ```bash
   apt-get -y update
   apt-get install -y gnupg2 pass docker-compose
   ```

2. Clone the repository:
   ```bash
   mkdir -p /src
   cd /src
   git clone git@github.com:eclipse-pass/pass-docker.git
   cd pass-docker
   git checkout minimal-assets
   ```

3. Run PASS:
   ```bash
   cd /src/pass-docker && \
     docker-compose pull && \
     docker-compose up
   ```

### PASS Release Process

PASS uses semantic versioning (`MAJOR.MINOR.PATCH`). The release process includes:

1. Code contribution
2. CI/CD via GitHub Actions
3. Building and testing
4. Generating release artifacts (Java artifacts and Docker images)
5. Publishing artifacts to repositories
6. Triggering deployment via AWS SQS
7. Updating infrastructure with new artifacts
8. Generating release notes

#### GitHub Actions Release Workflow

The "Publish: Release All" workflow automates the release process:

1. Builds and tests components
2. Publishes Java artifacts
3. Builds and pushes Docker images
4. Creates GitHub Releases

For detailed configuration, refer to the [pass-complete-release.yml](https://github.com/eclipse-pass/main/blob/main/.github/workflows/pass-complete-release.yml) Actions workflow.

Please refer to [github-cicd.md](./github-cicd.md) for further information on its useage.

## Related Information

- [PASS main repository](https://github.com/eclipse-pass/main)
- [PASS Docker repository](https://github.com/eclipse-pass/pass-docker)

For further assistance or questions, please open an issue in the [PASS main repository](https://github.com/eclipse-pass/main/issues) or find us in the [PASS Slack](https://eclipse-pass.slack.com).