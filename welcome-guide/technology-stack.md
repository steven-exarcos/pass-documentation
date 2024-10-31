# Technology Stack

### Introduction

PASS is composed of four main components: PASS UI, Data Loaders, PASS Core, and Deposit Services. These components are integral to the application running, but are flexible enough that they can be stood up on their own. Built with an array of open-source technologies, PASS guarantees a sturdy, secure, and transparent framework for deployment across diverse environments.

PASS UI contains the Submission UI which is responsible for the researcher workflow to review their grants and create submissions to their designated repositories, all while adhering to the policies established by the PASS Core's policy engine. PASS UI interfaces with PASS Core through Shibboleth authentication to support a single sign-on (SSO) experience, facilitating secure and streamlined access to the system's core functionalities.

PASS Core is the central back-bone to the application which is responsible for orchestrating the entire researcher workflow by communicating with external services, managing the data layer, applying policy and metadata rules. Data pertaining to journals, grants, and publications is funneled into PASS Core via Data Loaders, where it is then managed by the Java Persistence API (JPA). PASS Core also employs the Oxford Common File Layout (OCFL) for the storage of submission-related documents, offering options for disk or Amazon S3 bucket retention. When researchers are performing their submissions, it will flow through the PASS API. From there, a submission message is placed in a publication queue which the deposit services will pick up, assemble the deposit and transport it to their respective repository (DSpace, PubMed, etc).&#x20;

<figure>
    <img src="../.gitbook/assets/pass-architecture-simple-v2-wo-admin-ui.png" alt="">
    <figcaption>
        <p>PASS Architecture</p>
    </figcaption>
</figure>

### Front-end Technologies

To support the Submission UI the following technologies and frameworks are employed by PASS:

* [Ember.js](https://emberjs.com/): Selected for its robustness and opinionated framework structure, the Submission UI uses Ember.js to provide a clear, consistent, and easily to use workflow.&#x20;

### Back-end Technologies

The back-end is written in Java and uses the following technologies and frameworks:

#### Pass Core

* [Spring Boot](https://spring.io/projects/spring-boot): This is the back-bone of PASS, and the framework driving the development. Spring Boot simplifies the bootstrapping of PASS, by providing a methodology of convention over configuration for cleaner code and easier deployment.
* [Elide](https://elide.io/): Exposes a JSON-API web service, which enables a versatile and standardized API for communication to the core logic of PASS.
* [JPA](https://spring.io/projects/spring-data-jpa): The data access layer that communicates with the PostgresSQL database. Having Spring automatically wire up the interface to the database reduces boilerplate code and produces clean data access code.
* [OCFL](https://github.com/OCFL/ocfl-java): An open implementation in Java, OCFL is used within PASS' file service to reliably store documents. It has support for both filesystem storage and AWS S3 integration.
* [PostgreSQL](https://www.postgresql.org/): Serves as the database that stores all the information associated with grants, journals, publications, submissions, and policies.&#x20;

#### Data Loaders & Deposit Services

In addition to using Spring Boot the Data Loaders and Deposit Services use the following technologies:

* [SWORD](https://sword.cottagelabs.com/): This protocol is specifically utilized for the automated deposit of digital content into repositories. The SWORD protocol within PASS enables the standardized submission of scholarly works. In effect, providing compatibility and ease of integration with a variety of other repository platforms.
* [Amazon SQS](https://aws.amazon.com/sqs/): Queues the deposit requests, allowing for asynchronous processing and ensuring that submissions are handled reliably. This decouples the submission process from the deposit execution, enhancing system resilience and scalability.

### Deployment and Hosting

PASS is designed to be flexible and run on a variety of platforms; however at JHU we host PASS on Amazon Web Services. You can find more details about our deployment on our [Deployment Architecture](deployment-architecture.md) page.

### Development Tools and Practices

* [Ember CLI](https://cli.emberjs.com/release/): Offers a suite of tools to automate tasks such as building assets, running tests, and scaffolding code. It supports our front-end development of the Submission UI.
* [Docker](https://www.docker.com/): Utilized for running and testing PASS locally.
* [TestCafe](https://testcafe.io/): The main framework of our PASS acceptance tests. It is a free and open-source solution for running end-to-end tests.
* [Test Driven Development](https://en.wikipedia.org/wiki/Test-driven\_development) & [JUnit](https://junit.org/): The Eclipse PASS team follows test driven development (TDD) to ensure high code quality and build confidence. Additionally, we use JUnit for unit tests and [Spring Boot Test](https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#features.testing)/[Testcontainers](https://testcontainers.com/) for integration tests.&#x20;
* [GitHub](https://github.com/): GitHub is where the source code for PASS is hosted. GitHub is also used to trigger deployments using a feature called GitHub Actions.
