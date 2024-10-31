# Developer Documentation

The PASS Developer Documentation encompasses all aspects of PASS required to understand and contribute to the PASS
development. The main areas of PASS Development are PASS Core, PASS UI, Data Loaders, Deposit Services,
Notification Services, and Acceptance Tests. All of these components work together to bring together a comprehensive
system to disseminate research to their proper repositories. Each of these modules has a distinct role, such as managing
data ingestion, handling user interactions, automating data transformations, and making deposits to their intended 
downstream repository. In all, these components interact to create an efficient workflow that supports research 
dissemination in various institutional and federal compliance scenarios.

The Data Loaders handle data ingestion from external systems like PubMed Central, NIH, FIBI 
(JHU Grant Management System), transforming grant, journal, and manuscript submission data into a standardized format 
within PASS. Deposit Services then manage the packaging and transfer of these submissions to downstream repositories 
such as Pubmed Central and institutional repositories. Notification Services provide alerts and updates to relevant 
stakeholders based on submission workflows and events. This modular structure, built primarily using Java and Spring 
Boot, allows for flexibility and extensibility in accommodating different institutional requirements and third-party 
integrations.

The developer documentation also details the technical setup, configuration, and deployment of each PASS component, 
including environment-specific configurations, database setup, authentication management, and integration with cloud 
services provided by AWS. PASS utilizes RESTful APIs for data access and manipulation, while its microservices 
communicate asynchronously using message queues to support scalable and distributed deployments. By following a modular
and loosely-coupled architecture, PASS is able to evolve rapidly, incorporating new compliance requirements and
enhancing repository deposit capabilities across academic and research institutions.

**Table of Contents:**

1. [Use cases](use-cases.md)
2. [PASS Core](pass-core/README.md)
3. [PASS UI](pass-ui/README.md)
4. [Data Loaders](data-loaders/README.md)
5. [Deposit Services](deposit-service/README.md)
6. [Notification Services](notification-service/README.md)
7. [Pass Acceptance Testing](pass-acceptance-testing/README.md)
8. [PASS Docker](pass-docker/README.md)
9. [Release](release/README.md)