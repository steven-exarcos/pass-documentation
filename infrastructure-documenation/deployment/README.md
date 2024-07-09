# Deployment.md

---

## Overview

This guide covers the deployment architecture and release process for the PASS (Public Access Submission System) application. PASS is designed to be flexible and can adapt to various architectures, including cloud infrastructure, hybrid, or on-premises environments.


## Technical Deep Dive

excerpt from [../welcome-guide/deployment-architecture.md]

### Deployment Architecture

The deployment workflow starts when developers contribute code to the Eclipse PASS Git repository. Changes in this repository trigger GitHub Actions workflows, which are part of an automated CI/CD pipeline facilitating the deployment of the updated code. In deployment SQS is utilized for initiating the deployment from a GitHub workflow that publishes a SNS topic to the queue. The PASS deployment files contain configurations and environment variables that assist in the deployment of the PASS application and supporting services. Liquibase is utilized to manage database schema changes, which interacts with an AWS RDS instance running PostgreSQL.&#x20;

For other organizations looking to adopt a similar AWS application and deployment model, it's important to recognize that while the core architecture offers a template, it should be adapted to meet an organization's own requirements and needs. Each organization will need to evaluate its own application demands, data sensitivity, and user base to tailor the cloud resources, network configurations, and security policies accordingly. Moreover, integrating other types of cloud infrastructure or even on-premise solutions might be necessary to address specific technological preferences or regulatory requirements. Hybrid cloud environments or multi-cloud strategies could be employed to leverage the strengths of various cloud providers, enhance resilience, and avoid vendor lock-in. PASS is designed to be flexible and can adapt to a variety of architectures; whether a cloud infrastructure, hybrid or on-premise.

---

excerpt from [../docs/infra/digitalocean.md]

## Deployment Infrastructure

The current PASS infrastructure is running in Amazon Web Service (AWS). It includes the following core components in its stack:

* EC2 - Amazon Elastic Compute Cloud.

* ECS - Amazon Elastic Container Service.

* RDS - Amazon Relational Database Service.

* S3 - Amazon S3.

* ALB - Amazon Application Load Balancer.

* WAF - AWS Web Application Firewall.

## Provisioning PASS

### Installing PASS Dependencies

#### VM/EC2 Instructions Using Docker Compose

On the server, you will need [docker / docker-compose](https://docs.docker.com/compose/) and [pass-docker](https://github.com/eclipse-pass/pass-docker)


```bash
apt-get -y update
apt-get install -y gnupg2 pass docker-compose
mkdir -p /src
cd /src
git clone git@github.com:eclipse-pass/pass-docker.git
cd pass-docker
git checkout minimal-assets
```

### Runing PASS

#### VM/EC2 Instructions Using Docker Compose
```bash
cd /src/pass-docker && \
  docker-compose pull && \
  docker-compose up
```

---
excerpt from [../docs/dev/release.md]

## Release Process

A PASS release produces a set of Java artifacts, and Docker images.

Each release of PASS has its own version which is used by every component. PASS uses `MAJOR.MINOR.PATCH` [semantic versioning](https://semver.org/) approach. The PASS Docker environment is updated with references to the released images.
Source code is tagged and release notes made available.

The PASS release process involves the following steps:

Code Contribution : Developers contribute code changes to the PASS Git repository.

Continuous Integration (CI) : Changes in the repository trigger GitHub Actions workflows, which are part of an automated CI/CD pipeline.

Build and Test : The CI pipeline builds and tests the PASS application, ensuring code quality and functionality.

Release Artifacts : Upon successful testing, the pipeline generates release artifacts, including Java artifacts and Docker images.

Artifact Publishing : Java artifacts are published to Sonatype Nexus and Maven Central repositories, while Docker images are pushed to the GitHub Container Registry.

Deployment Trigger : The CI pipeline publishes a message to an Amazon Simple Queue Service (SQS) queue, initiating the deployment process.

Infrastructure Updates : The deployment process updates the PASS infrastructure with the latest release artifacts, including updating the ECS cluster with the new Docker images.

Release Notes : Release notes are generated and made available to stakeholders.


---
excerpt from [../docs/dev/release-steps-with-automation.md]

#### GitHub Actions
.

##### Workflow: Release All Projects

The [Publish: Release All](/.github/workflows/pass-complete-release.yml) is a GitHub workflow that will release all the PASS projects in the correct order, build and push docker images, and create GitHub Releases.

This workflow performs the following tasks:

1. Checks out the PASS repository code.

2. Sets up the build environment (e.g., Java, Docker).

3. Builds and tests the PASS application components.

4. Publishes Java artifacts to Sonatype Nexus and Maven Central.

5. Builds and pushes Docker images to the GitHub Container Registry.

6. Creates GitHub Releases with release notes and artifacts.

# Example snippet from the "Publish: Release All" workflow
jobs:
  release:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Set up JDK
        uses: actions/setup-java@v3
        with:
          java-version: '11'
          distribution: 'adopt'
      - name: Build and Release
        run: |
          ./mvnw clean install
          ./mvnw deploy -Prelease
      - name: Build and Push Docker Images
        run: |
          docker build -t pass/app:${{ github.sha }} .
          docker push pass/app:${{ github.sha }}
      - name: Create GitHub Release
        uses: actions/create-release@v1
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        with:
          tag_name: v${{ github.ref }}
          release_name: Release ${{ github.ref }}
          body: ${{ steps.release-notes.outputs.content }}

For more detailed information and configuration, refer to the pass-complete-release.yml file in the AWS-PASS-Deploymnet repository.

## Additional Resources

PASS Documentation

PASS Docker Repository

For further assistance or questions, please _______.