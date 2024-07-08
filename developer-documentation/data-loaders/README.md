# PASS Data Loaders

The PASS Data Loaders are comprised of three components in the [Pass Support](https://github.com/eclipse-pass/pass-support) repository: [Pass Journal Loader](https://github.com/eclipse-pass/pass-support), [Pass NIHMS Loader](https://github.com/eclipse-pass/pass-support), [Pass Grant Loader](https://github.com/eclipse-pass/pass-support). These three components are responsible for loading external data into PASS.

# Summary

All three loaders are Java JAR command line applications and can be run from any platform that can run Java applications. These application utilize system properties and can be configured to run with different parameters.

## [Grant Loader](./grant-loader.md)

The Grant Loader is designed to automate the ingestion and processing of grant data from various sources into PASS. It handles the loading of grants using predefined configurations and mappings, ensuring the correct representation and association of grant data. Since there are predefined fields that are required to represent a grant and institutions may have varying representations, specific implementations may be needed in order to accommodate other institutions. The design of the Grant Loader is flexible in that it can accommodate development of connectors to varying data sources.

## [Journal Loader](./journal-loader.md)

The Journal Loader facilitates the automated loading of journal data, in particular from sources such as [PubMed](https://pubmed.ncbi.nlm.nih.gov/) and [PubMed Central](https://www.ncbi.nlm.nih.gov/pmc/). The Journal Loader is responsible for streamlining the process of updating and maintaining journal entries in PASS.

## [NIHMS Loader](./nihms-loader.md)

The NIHMS Loader specifically targets the loading and transformation of manuscript submission data from the NLM’s Public Access Compliance Monitor (PACM) into PASS. This enables publications in PASS to be updated appropriately with their publication information that is in PubMed Central. The PACM [user guide](https://www.ncbi.nlm.nih.gov/pmc/utils/pacm/static/pacm-user-guide.pdf) explains the background of the PACM system and data. It will assist in setting up the appropriate accounts in order to access the PACM system and API.

# Technologies Utilized

- [Docker](https://www.docker.com/products/docker-desktop/) for running and testing the applications.
- [Java 17+](https://www.oracle.com/java/technologies/downloads/) for the application development.
- [Spring Boot](https://spring.io/projects/spring-boot) for the application framework.

# Related Information

The following resources cover external resources for data extraction and the infrastructure used to run the Data Loaders:

- PubMed Central Resources
    - [PACM Guide](https://www.ncbi.nlm.nih.gov/pmc/utils/pacm/static/pacm-user-guide.pdf)
    - [PMC APIs](https://www.ncbi.nlm.nih.gov/pmc/tools/developers/#pmc-apis)
- NIHMS Resources
    - [NIH Manuscript Submission](https://www.nihms.nih.gov)
    - [NIH Manuscript Submission Process](https://www.nihms.nih.gov/about/overview/)
- PASS
  - [PASS Architecture](../../welcome-guide/deployment-architecture.md)
  - [PASS Support](https://github.com/eclipse-pass/pass-support)