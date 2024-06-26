# Journal Loader

The Journal Loader is responsible for pulling data from [PubMed Central](https://www.ncbi.nlm.nih.gov/pmc/) (PMC) and the [MEDLINE database](https://www.nlm.nih.gov/medline/medline_home.html), and making the appropriate updates to PASS.

# Journal Loader Summary

The Journal Loader parses the PMC type A journal `.csv` file, and/or the [MEDLINE database](https://www.nlm.nih.gov/medline/medline_home.html) `.txt` file, and syncs with the repository, by taking the following actions:

- Adds journals if they do not already exist
- Updates PMC method A participation if it differs from the corresponding resource in the repository

# Knowledge Needed / Skills Inventory

- Development of the Journal Loader
    - Programming in Java
    - Basic understanding of NLM journal loader
- Running the Journal Loader
    - CLI commands

# Technologies Utilized

- [Java 17+](https://www.oracle.com/java/technologies/downloads/)
- [Spring Boot](https://spring.io/projects/spring-boot)

# Technical Deep Dive

## Usage

Using java system properties to launch the journal loader. Note: update the version of the jar name to the one that is being used.

```shell
java -Dpmc=https://www.ncbi.nlm.nih.gov/pmc/front-page/NIH_PA_journal_list.csv -Dmedline=https://ftp.ncbi.nih.gov/pubmed/J_Medline.txt -Dpass.core.url=http://localhost:8080 -Dpass.core.user=USER -Dpass.core.password=PASS -jar pass-journal-loader-nih/target/pass-journal-loader-nih-1.8.0-SNAPSHOT-exe.jar
```

### Properties or Environment Variables

The following may be provided as system properties on the command line `-Dprop-value`.

`pass.core.url`
The base url for the pass-core REST API such as `http://localhost:8080`

`pass.core.user`
The pass-core backend user.

`pass.core.password`
The pass-core backend user password.

`dryRun`
Do not add or update resources in the repository, just give statistics of resources that would be added or updated

`pmc`
URL of the PMC "type A" journal .csv file, for example
[https://www.ncbi.nlm.nih.gov/pmc/front-page/NIH_PA_journal_list.csv](https://www.ncbi.nlm.nih.gov/pmc/front-page/NIH_PA_journal_list.csv)

`medline`
URL of the Medline journal file, for example
[https://ftp.ncbi.nih.gov/pubmed/J_Medline.txt](https://ftp.ncbi.nih.gov/pubmed/J_Medline.txt)

`LOG.*`
Adjust the logging level of a particular component, e.g. `LOG.org.eclipse.pass=WARN`

## Journal Loader Classes & Data Flow Overview

### Data Flow

1. Initialization:
    - The `Main` class initializes the application and calls the `BatchJournalFinder` and `LoaderEngine` to start processing.
2. File Processing:
    - `BatchJournalFinder` processes each file using the appropriate reader (`MedlineReader`, `NihTypeAReader`).
      - The `load` method in `BatchJournalFinder` initiates the process, collects files to be processed.
3. Data Loading:
    - Processed journal data is passed to `LoaderEngine` to be loaded into the target system.
      - If a journal is not found then a new one will be created, otherwise it will update the journal.


# Next Step / Institution Configuration

Journal loader is simple to configure. It will run on any system that can run Java applications and doesn't need any external account setup. The two sources of data PMC Type A Journals and MEDLINE do not require accounts to access the data. Like both the [NIHMS Loader](./nihms-loader.md) and [Grant Loader](./grant-loader.md), the Journal Loader is run using [AWS Batch and ECS](../../welcome-guide/deployment-architecture.md#pass-deployment-architecture).

# Related Information

The following resources are the sources of the journal information that is loaded into pass:

- [PubMed Central](https://www.ncbi.nlm.nih.gov/pmc/)
- [NLM MEDLINE](https://www.nlm.nih.gov/medline/medline_overview.html)