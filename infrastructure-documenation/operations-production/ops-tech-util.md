# Operations/Production - Technology Utilized

The PASS production infrastructure and operations are composed of a variety of technologies. The chosen technology stack
was desirable for its ease of setup, support for CI/CD, and scalability. 

## Technologies Utilized

- [Amazon Web Services (AWS) Cloud Computing](https://aws.amazon.com): Many webservices are use for the majority of the
infrastructure including virtual servers (EC2), Amazon Relational Database service (RDS, using a PostgreSQL instance),
Elastic Cloud Compute (ECS), and a variety of others.
- [AWS Account Management](https://aws.amazon.com/account/): Manages users and the policies around the resources that
users will use.
- [AWS Command Line Interface](https://aws.amazon.com/cli/): Used in the development, testing, and management of the JHU
PASS infrastructure.
- [Docker](https://www.docker.com/): Docker is used in all our environments for running the main components of the PASS
application.
- [GitHub Actions](https://docs.github.com/en/actions): GitHub Actions are used for deployment and CI/CD operations.
