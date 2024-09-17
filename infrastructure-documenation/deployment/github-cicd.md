# GitHub CI/CD Guide

## Summary

GitHub Actions is a powerful [CI/CD](./github-cicd.md) platform integrated directly into GitHub repositories. It allows you to automate various software development workflows, including building, testing, and [deploying](./README.md) your code.

### Key Concepts

1. **Workflows**: YAML files that define a set of jobs to be executed when triggered by an event.
2. **Jobs**: A set of steps that execute on the same runner.
3. **Steps**: Individual tasks that can run commands or actions.
4. **Actions**: Reusable units of code that can be shared across workflows.
5. **Events**: Specific activities that trigger a workflow run.

### Workflow Action Structure

```yaml
name: CI

on: [push, pull_request]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - name: Run a script
        run: echo Hello, world!
```

This simple workflow runs on push and pull request events, checks out the repository, and runs a simple command.

## GitHub Secrets

GitHub secrets are encrypted environment variables used to store sensitive information securely. They are crucial for handling authentication and other confidential data in your workflows.

### Types of Secrets

1. **Organization Secrets**: Available to all repositories in the `eclipse-pass` organization.
2. **Repository Secrets**: Specific to a single repository.
3. **Environment Secrets**: Tied to a specific environment within a repository.

### Creating Secrets

Due to permission restrictions, PASS project members should use the provided Python script to create repository or environment secrets:

```bash
python github_secrets.py -u <username> -t <token> -r <repo> -n <name> -v <value> [-e <environment>]
```

For organization secrets, open a ticket with the [Eclipse Help Desk](https://gitlab.eclipse.org/eclipsefdn/helpdesk).

### Using Secrets in Workflows

Reference secrets in your workflows like this:

```yaml
${{ secrets.SECRET_NAME }}
```

For reusable workflows, pass secrets explicitly:

```yaml
jobs:
  call-publish-docker:
    uses: ./.github/workflows/docker-publish.yml
    secrets:
      AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
      AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
```

In the called workflow, declare expected secrets:

```yaml
on:
  workflow_call:
    secrets:
      AWS_ACCESS_KEY_ID:
        required: true
      AWS_SECRET_ACCESS_KEY:
        required: true
```
### Best Practices

1. Use reusable workflows for common tasks to maintain DRY principles.
2. Leverage GitHub-hosted runners when possible to reduce maintenance overhead.
3. Use environment protection rules for sensitive deployments.
4. Regularly audit and rotate your secrets.
5. Use GitHub Actions marketplace for pre-built actions to speed up development.

## AWS Integration

To interact with AWS services, including ECR (Elastic Container Registry), you'll need to set up appropriate secrets and use AWS-specific actions in your workflows.

### Setting up AWS Credentials

Store your AWS credentials as secrets:

1. `AWS_ACCESS_KEY_ID`
2. `AWS_SECRET_ACCESS_KEY`

### Example Workflow for AWS Deployment

```yaml
name: Deploy to ECR

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      
      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v1
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: us-east-1

      - name: Login to Amazon ECR
        id: login-ecr
        uses: aws-actions/amazon-ecr-login@v1

      - name: Build, tag, and push image to Amazon ECR
        env:
          ECR_REGISTRY: ${{ steps.login-ecr.outputs.registry }}
          ECR_REPOSITORY: my-ecr-repo
          IMAGE_TAG: ${{ github.sha }}
        run: |
          docker build -t $ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG .
          docker push $ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG
```

This workflow builds a Docker image and pushes it to Amazon ECR.

## Related Information

For more detailed information, refer to the [GitHub Actions documentation](https://docs.github.com/en/actions).