# NIHMS Loader

The NIH Manuscript Submission Loader (NIHMS Loader) contains the components required to download, transform, and load Submission information from NIHMS to PASS. 

## Summary

The NIHMS Loader is a module contained in [Pass-Support](https://github.com/eclipse-pass/pass-support), and is composed of two Java command line tools. The first uses the NIH API to download the CSV(s) containing compliant, non-compliant, and in-process publication information. The second tool reads those files, transforms the data to the PASS data model, and then loads them to PASS.

For background information on the NIH Public Access Compliance Monitor (PACM), see the [user guide](https://www.ncbi.nlm.nih.gov/pmc/utils/pacm/static/pacm-user-guide.pdf). Limited information on the API is provided by the [NLM Technical Bulletin](https://www.nlm.nih.gov/pubs/techbull/mj19/brief/mj19_api_public_access_compliance.html)

The NIHMS Loader operates in two stages:

* Harvests data from the NLM’s Public Access Compliance Monitor (PACM) website about the compliance status of publications written by researcher PIs
* It compares PACM data with `Submission` data in PASS and then adds or updates `Submission`, `Publication`, and `RepositoryCopy`.

These two processes are separate Java command line interface (CLI) applications: the NIHMS Data Harvest CLI and the NIHMS Transform and Load CLI.

## Knowledge Needed / Skills Inventory

* Development of the NIHMS Loader
  * Programming in Java
  * Basic understanding of PMC data
  * REST/HTTP
* Running the NIHMS Loader
  * CLI commands
  
## Technologies Utilized

* [Docker](https://www.docker.com/products/docker-desktop/)
* [Java 17+](https://www.oracle.com/java/technologies/downloads/)
* [Spring Boot](https://spring.io/projects/spring-boot)

## Technical Deep Dive

### NIHMS Data Harvest CLI

The NIHMS Data Harvest CLI uses the NIH API to download the PACM data.

The following are required to run this tool:

* Java 17+
* Download the latest [docker image](https://github.com/eclipse-pass/pass-support/pkgs/container/pass-nihms-loader) or download the latest [pass-support release](https://github.com/eclipse-pass/pass-support/releases) and compile the `pass-nihms-loader` module
* An account for the NIH PACM website, and obtain an API key. The API key is only valid for 3 months, so it will need to be updated periodically. There are a couple of ways to obtain an account, and the process is institution-specific.

### Data Harvest Configuration

There are several ways to configure the Data Harvest CLI. Data Harvest CLI is a Spring Boot Application, so it can be configured using [Spring Boot Configuration](https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features.external-config)

You will need to set values for the following properties: 
- `nihmsetl.api.url.param.inst`
- `nihmsetl.api.url.param.ipf`
- `nihmsetl.api.url.param.api-token`

The full set of properties for the NIHMS Harvester is listed below. These properties are set in the `resources/application.properties` file in the `nihms-data-harvest` module.

| Property                           | Default Value          | Notes                                                                                                                                                                                      |
|:-----------------------------------|:-----------------------|:-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| nihmsetl.data.dir                  | /data/nihmsloader/data | Defines a directory where the files will be downloaded. If not specified, the default directory will be created for you.                                                                   |
| nihmsetl.api.host                  | www.ncbi.nlm.nih.gov   | The host name of the API. The default should not change. If the API is moved to another host this will be updated by the PASS team.                                                        |
| nihmsetl.api.scheme                | https                  | The HTTP scheme of the API. The default should not change.                                                                                                                                 | 
| nihmsetl.api.path                  | /pmc/utils/pacm/       | The API URL Path. The default should not change. If the API does change the URL, the default value will be updated by the PASS team.                                                       |
| nihmsetl.http.read-timeout-ms      | 30000                  | Allow 30 seconds for a request to be read before timing out.                                                                                                                               |
| nihmsetl.http.connect-timeout-ms   | 30000                  | Allow 30 seconds for establishing connections before timing out.                                                                                                                           |
| nihmsetl.api.url.param.format      | csv                    | The format for the downloaded file. The default should not change. The NIHMS Transform and Load requires a csv file.                                                                       |
| nihmsetl.api.url.param.inst        |                        | Name of the institution making the API request. The value for your organization can be found in the [PACM website](https://www.ncbi.nlm.nih.gov/pmc/utils/pacm).                           |
| nihmsetl.api.url.param.ipf         |                        | IPF (Institutional Profile File) number, the unique ID assigned to a grantee organization in the eRA system.                                                                               |
| nihmsetl.api.url.param.api-token   |                        | The API token retrieved from the [PACM website](https://www.ncbi.nlm.nih.gov/pmc/utils/pacm). The token expires every three months.                                                        |
| nihmsetl.api.url.param.pdf         | See Notes              | Date in MM/YYYY format that the PACM data should start from. Can be set using the `-s` harvester command line option). By default this date will be set to the current month, one year ago |
| nihmsetl.api.url.param.pdt         | See Notes              | Date in MM/YYYY format that the PACM data should end. Leave blank to default to the current month.                                                                                         |

### Running the Data Harvester

The simplest way to run the data harvester is to use the java command in a CLI with [Java system property option](https://docs.oracle.com/en/java/javase/17/docs/specs/man/java.html), by using the -D parameter. Note if you're running on Windows you may have double quote the parameters, for example `"-Dnihmsetl.api.url.param.inst=my-inst"`.

```shell
java -Dnihmsetl.api.url.param.inst=my-inst -Dnihmsetl.api.url.param.ipf=my-ipf -Dnihmsetl.api.url.param.api-token=my-token -jar nihms-data-harvest-cli-exec.jar
```

Once the Data Harvest CLI has been configured, there are a few additional options you can add when running from the command line.

By default, all 3 publication statuses - compliant, non-compliant, and in-process CSVs will be downloaded. To download one or two of them, you can add them individually at the command line:

```text
-c, -compliant, --compliant - Download compliant publication CSV.
-p, -inprocess, --inprocess - Download in-process publication CSV.
-n, -noncompliant, --noncompliant - Download non-compliant publication CSV.
```

You can also specify a start date, by default the PACM system sets the start date to 1 year prior to the date of the download. You can change this by adding a start date parameter. This will return all records published since the date provided. The syntax for this parameter is mm-yyyy .

```text
-s, -startDate --startDate
```

So, for example, to download the compliant publications published since December 2012, you would do the following:

```shell
java -jar nihms-data-harvest-cli-exec.jar -s 12-2012 -c
```

On running this command, files will be downloaded and renamed with a prefix according to their status ("compliant", " noncompliant", or "inprocess") and a timestamp integer e.g. noncompliant_nihmspubs_20180507104323.csv.

### NIHMS Transform and Load

The NIHMS Data Transform-Load CLI reads data in from CSVs that were downloaded from the PACM system, converts them to PASS compliant data and loads them into the PASS database.
Pre-requisites

The following is required to run this tool:

* Java 17+
* Download the latest [docker image](https://github.com/eclipse-pass/pass-support/pkgs/container/pass-nihms-loader) or download the latest [pass-support release](https://github.com/eclipse-pass/pass-support/releases) and compile the `pass-nihms-loader` module. The jar to run for this process is `nihms-data-transform-load-exec.jar` found in the `nihms-data-transform-load` module.

### Data Transform-Load Configuration

There are several ways to configure the Data Transform-Load CLI. Data Transform-Load CLI is a Spring Boot Application, so it can be configured as described here: Spring Boot Configuration

You will need to set values for the following props: nihmsetl.repository.id, pass.client.url, pass.client.user and pass.client.password. There are a number of ways to do this with a Spring Boot app, which are described in the link above.

Here is an example using Java system properties -D.

> java -Dnihmsetl.repository.id=my-repo-id -Dpass.client.url=my-url -Dpass.client.user=my-user -Dpass.client.password=my-pw -jar nihms-data-transform-load-exec.jar

### Running the Data Transform-Load

Once the Data Transform-Load CLI has been configured, there are a few additional options you can add when running from the command line.

By default, all 3 publication statuses: `compliant`, `non-compliant`, and `in-process` CSVs will be downloaded. To download one or two of them, you can add them individually at the command line:

    -c, -compliant, --compliant
    -p, -inprocess, --inprocess
    -n, -noncompliant, --noncompliant

So, for example, to process non-compliant spreadsheets only:

```shell
java -jar nihms-data-transform-load-cli-exec.jar -n
```

When run, each row will be loaded into the application and new Publications, Submissions, and RepositoryCopies will be created in PASS as needed. The application will also update any Deposit.repositoryCopy links where a new one is discovered. Once a CSV file has been processed, it will be renamed with a suffix of ".done" e.g. noncompliant_nihmspubs_20180507104323.csv.done. To re-process the file, simply rename it to remove the .done suffix and re-run the application.

### Running the Harvester and Data Transform-Load using Docker

To run both the Harvest and Data Transform-Load using Docker you will also need [PASS docker](../../welcome-guide/setup-run-pass.md) running, otherwise it will fail on the Transform-Load step.

Once PASS Docker is running, use docker pull to get the Pass NIHMS Loader image, and be sure to replace the image tag `1.8.0-snapshot` with the version that you want to pull and run.

```shell
docker pull ghcr.io/eclipse-pass/pass-nihms-loader:1.8.0-snapshot
```

Run the docker image with the following environment variables, using the [docker -e parameter](https://docs.docker.com/reference/cli/docker/container/run/). On Windows you may have double quote the parameters, for example `"-eNIHMS_API_INST=YOUR_INST"`.

```shell
docker run -eNIHMS_API_INST=YOUR_INST -eNIHMS_API_IPF=YOUR_IPF -eNIHMS_API_TOKEN=YOUR_API_TOKEN ghcr.io/eclipse-pass/pass-nihms-loader:1.8.0-snapshot
```

### NIHMS Data Harvester Classes and Relationships

#### Data Flow Overview

* **Initialization**:
    * `NihmsHarvesterCLIRunner` starts and processes command-line arguments.
    * `NihmsHarvesterCLI` configures and initializes `NihmsHarvester` using `NihmsHarvesterConfig`.
* **URL Construction**:
    * `NihmsHarvester` uses `UrlBuilder` and `UrlType` to create URLs needed to access NIHMS data.
* **Data Download**:
  * `NihmsHarvesterDownloader` retrieves data from the constructed URLs.
  * Downloaded data is passed back to `NihmsHarvester`.
* **Data Processing**:
    * `NihmsHarvester` processes the downloaded data.

#### Interactions and Dependencies

* `NihmsHarvester` is the central class, relying on configurations from `NihmsHarvesterConfig`, URL construction from `UrlBuilder` and `UrlType`, and data downloading from `NihmsHarvesterDownloader`.
* `NihmsHarvesterCLI` and `NihmsHarvesterCLIRunner` are entry points for the application, primarily handling user interaction and delegation to the core `NihmsHarvester`.
* `UrlBuilder` and `UrlType` ensure that the URLs used for data retrieval are correctly constructed and categorized.

### NIHMS Data Transform-Load Classes and Relationships

#### Data Flow Overview

* **Data Transformation and Loading**:
  * `NihmsTransformLoadCLIRunner` and `NihmsTransformLoadCLI` handle command-line interactions for data transformation and loading.
  * `NihmsTransformLoadService` transforms publication data and loads it using `SubmissionLoader`.
  * `PmidLookup` service to retrieve a PMID record from the [NBCI Entrez API](https://www.ncbi.nlm.nih.gov/books/NBK25497/). 
* **Data Transfer**:
  * `SubmissionDTO` encapsulates transformed submission data for transfer between components.
  * `NihmsPassClientService` service to provide interactions with the data via the PASS client.
  * `SubmissionLoader` loads the final submission data into PASS and accomplishes this by interfacing with the `NihmsPassClientService`.

#### Interactions and Dependencies

* `NihmsTransformLoadCLI` is the entry point and passes control to `NihmsTransformLoadCLIRunner`.
* `NihmsTransformLoadCLIRunner` initializes the process using configurations from `NihmsTransformLoadConfig` and performs the transformation and loading by invoking `NihmsTransformLoadService`.
* `NihmsTransformLoadConfig` provides necessary configuration settings to `NihmsTransformLoadService`.
* `NihmsTransformLoadService` orchestrates the transformation of data using `NihmsPublicationToSubmission` and the loading of data using `SubmissionLoader`.
* `NihmsPublicationToSubmission` is responsible for transforming PMC publication data and associated data to a `SubmissionDTO`, which is composed of `Grant`, `Publication`, `RepositoryCopy`, and `Submission` objects.
* `SubmissionDTO` acts as a container for the transformed data.
* `SubmissionLoader` is responsible for the final step of loading the data into PASS.

## Next Step / Institution Configuration

Configuring the NIHMS loader to run at an institution requires several NIH/NLM accounts to be setup. The first step would be to familiarize yourself with the [PACM Guide](https://www.ncbi.nlm.nih.gov/pmc/utils/pacm/static/pacm-user-guide.pdf). Institutions that are universities typically have an Office of Research and that is a good starting point to finding out more information how PACR roles are assigned, typically a Signing Officer at the Office of Research can perform this function.

Once access to PACM has been established, configuring the infrastructure where the Data Harvester and the Data Transform-Load applications is the next step. Since they are Java applications they can be scheduled as cron jobs on a server or run in the cloud. At Johns Hopkins University these applications are run by using [AWS Batch and ECS](../../welcome-guide/deployment-architecture.md#pass-deployment-architecture).

Optionally as a last step, setting up the [NIH Manuscript Submission](https://www.nihms.nih.gov) account would enable seeing which submissions have been made and how far along they are in the process. Once a submission makes its way through this process it appears in the PubMed Central data. This can be useful for troubleshooting records expected to be in the PubMed Central data.

### Manual NIHMS Token Generation

Once a PACM account is established, generating the token can be done via the [PACM Website](http://www.pubmedcentral.nih.gov/utils/pacm/). Use the eRA Commons login and sign in with your username and password. Note that the login instructions may be different for every institution as the authentication mechanism may differ. After logging in, use the link at the top of the page that is labeled `API Token`. This page will generate a new token every time it is accessed and your new API token will appear here. A token is valid for three months, and currently there is no way of deleting a token, so each page access will create a new token.

### NIHMS Token Refresh

The NIHMS harvester process requires an Authentication token. This token is available from the PACM Utils page and is valid for three months. There is currently no API available to refresh the token.

In order to provide an automatic token refresh, there is a module within the NIHMS loader called `nihms-token-refresh` that performs an automatic refresh of the token and updates the AWS Parameter Store. More information about how this module works in a production environment can be found in the [Operation/Production Data Loaders section](../../infrastructure-documenation/operations-production/ops-loaders.md#nihms-api-token-refresh-automation).

## Related Information

The resources below will assist in setting up the accounts required to run the NIHMS Loader and understanding the submission process.

* PubMed Central Resources
  * [PACM Guide](https://www.ncbi.nlm.nih.gov/pmc/utils/pacm/static/pacm-user-guide.pdf)
  * [PMC APIs](https://www.ncbi.nlm.nih.gov/pmc/tools/developers/#pmc-apis)
* NIHMS Resources
  * [NIH Manuscript Submission](https://www.nihms.nih.gov)
  * [NIH Manuscript Submission Process](https://www.nihms.nih.gov/about/overview/)