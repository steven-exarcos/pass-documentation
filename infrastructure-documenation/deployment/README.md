# Deployment.md

---

## Overview

This guide covers the deployment architecture and release process for the PASS (Portable Application Security Solution) application. PASS is designed to be flexible and can adapt to various architectures, including cloud infrastructure, hybrid, or on-premises environments.


## Technical Deep Dive

excerpt from [../welcome-guide/deployment-architecture.md]
### Deployment Architecture

The deployment workflow starts when developers contribute code to the Eclipse PASS Git repository. Changes in this repository trigger GitHub Actions workflows, which are part of an automated CI/CD pipeline facilitating the deployment of the updated code. In deployment SQS is utilized for initiating the deployment from a GitHub workflow that publishes a SNS topic to the queue. The PASS deployment files contain configurations and environment variables that assist in the deployment of the PASS application and supporting services. Liquibase is utilized to manage database schema changes, which interacts with an AWS RDS instance running PostgreSQL.&#x20;

For other organizations looking to adopt a similar AWS application and deployment model, it's important to recognize that while the core architecture offers a template, it should be adapted to meet an organization's own requirements and needs. Each organization will need to evaluate its own application demands, data sensitivity, and user base to tailor the cloud resources, network configurations, and security policies accordingly. Moreover, integrating other types of cloud infrastructure or even on-premise solutions might be necessary to address specific technological preferences or regulatory requirements. Hybrid cloud environments or multi-cloud strategies could be employed to leverage the strengths of various cloud providers, enhance resilience, and avoid vendor lock-in. PASS is designed to be flexible and can adapt to a variety of architectures; whether a cloud infrastructure, hybrid or on-premise.

---
excerpt from [../docs/dev/release.md]
## Release Process

A PASS release produces a set of Java artifacts, and Docker images.
Java artifacts are pushed to Sonatype Nexus and Maven Central repositories. Docker images are pushed to GitHub Container Registry.

The PASS Docker environment is updated with references to the released images.
Source code is tagged and release notes made available.

Each release of PASS has its own version which is used by every component. PASS uses `MAJOR.MINOR.PATCH` [semantic versioning](https://semver.org/) approach.

---
excerpt from [../docs/dev/release-steps-with-automation.md]
#### GitHub Actions Workflow
.

##### Release All Projects

The [Publish: Release All](/.github/workflows/pass-complete-release.yml) is a GitHub workflow that will release all the PASS projects in the correct order, build and push docker images, and create GitHub Releases.

---
excerpt from [../docs/infra/digitalocean.md]
## Infrastructure 

The current PASS infrastructure includes the following components:

* EC2

* ECS

* RDS

* S3

* ALB

* WAF

#### Installing PASS Dependencies

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

#### Running PASS

```bash
cd /src/pass-docker && \
  docker-compose pull && \
  docker-compose up
```


