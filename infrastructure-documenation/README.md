# Infrastructure Documentation

The PASS project's infrastructure documentation provides a comprehensive overview of the practices, deployment guides, 
and operational strategies that support the development, delivery, and maintenance of the system. The team employs a
Continuous Integration and Continuous Delivery (CI/CD) strategy leveraging GitHub Actions which automates our testing
and deployment workflows. Currently, the team is focused on reducing manual interventions within the pipeline, 
moving towards a fully automated continuous deployment model. 

The deployment guide details how to set up the PASS platform across various environments, including cloud infrastructure
on AWS, using components like EC2, ECS, RDS, and S3. It emphasizes the use of infrastructure-as-code tools such as 
Docker Compose, and details the deployment workflow. It includes a roadmap for the future dev-ops technologies like 
Kubernetes, ArgoCD/Flux, Prometheus/Grafana.

In the operations and production section, the documentation details the architecture and infrastructure management 
strategies implemented by JHU to monitor, maintain, and optimize the platform. This assists the JHU team in their own
documentation of their production environment, but also serves as one example of a production environment for other 
adopters of PASS. This includes AWS-based deployment models, monitoring, data management, and ensuring consistent 
operations across environments.

**Table of Contents:**

1. [CI/CD](ci-cd/README.md)
2. [Deployment](deployment/README.md)
3. [Operations/Production](operations-production/README.md)