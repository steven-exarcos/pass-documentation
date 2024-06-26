# Grant Loader

The Grant Loader ingests grant data from an institution and maps the data to the appropriate PASS Objects in the PASS Data Model.

## Summary

This module comprises code for retrieving grant data from some kind of data source, and using that data to update the PASS backend. Typically, the data pull will populate a data structure, which will then be consumed by a loader class. While this sounds simple in theory, there are several considerations which may add to the complexity of implementations. An implementor must first determine data requirements for the use of the data once it has been ingested into PASS, and then map these requirements to data available from the data source. It may be that additional data from other services may be needed in order to populate the data structures to be loaded into PASS. On the loading side, the implementor may need to support different modes of ingesting data. Additional logic may be needed in the data loading apparatus to resolve the fields in the data assembled by the pull process. For example, we will need to consider that several systems may be updating PASS objects, and that other services may be more authoritative for certain fields than the service providing the grant data. The JHU implementation is complex regarding these issues.

## Knowledge Needed / Skills Inventory

- Development of the Grant Loader
    - Programming in Java
    - Basic understanding of your institutions grant data
- Running the Grant Loader
    - CLI commands

## Technologies Utilized

- [Docker](https://www.docker.com/products/docker-desktop/)
- [Java 17+](https://www.oracle.com/java/technologies/downloads/)
- [Spring Boot](https://spring.io/projects/spring-boot)

## Technical Deep Dive

### Configuring using Spring Boot Profiles

The grant loader uses Spring Boot Profiles to select the appropriate classes to be used for a given institution. There is a property in application.properties named `spring.profiles.active` that needs to be set when starting the grant loader.  This property can be set at runtime as well using the normal spring boot configuration functionality: [Spring Boot Configuration](https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features.external-config).

The code has been factored to ease development for multiple institutions. These are the institution-specific classes which typically need to be implemented and annotated with the `@Profile` annotation:

#### Connector

The Connector class connects to the data store for an institution's implementation, and operates on the data to supply, in as standard a form as possible, the data to be consumed by the Updater.

#### Updater

The Updater class takes the data supplied by the Connector and creates or updates the corresponding objects in the PASS repository accordingly. There is a Default class whose children may override certain substantive methods if the local policies require.

### Profiles

#### JHU

The JHU implementation is used to pull data from the COEUS/FIBI database views for the purpose of performing regular updates. We look at grants which have been updated since a particular time (typically the time of the previous update), join this with user and funder information associated with the grant, and then use this information to update the data in the PASS backend. The JHU implementation also treats the COEUS/FIBI database as authoritative for all fields in the data. If a grant is being passed in for update, it is assumed that all records for that grant are included in the input.

### Grant Data

In order for PASS to map grant data to the associated Objects within PASS, the Grant Loader needs to ingest a CSV file with the following fields and data types:

| Column                | Type/Size      | Required | Description                                                                                                                                                                                                                                   |
|-----------------------|----------------|----------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| GRANT_NUMBER          | TEXT/255       | Y        | Unique identifier for the grant (institutional Grant ID)                                                                                                                                                                                      |
| GRANT_TITLE           | TEXT/255       | Y        | The title of the grant                                                                                                                                                                                                                        |
| AWARD_NUMBER          | TEXT/255       | N        | Unique identifier for the award                                                                                                                                                                                                               |
| AWARD_STATUS          | TEXT/255       | N        | The status of the award. Valid values: active, pre-award, terminated                                                                                                                                                                          |
| AWARD_DATE            | DATE           | N        | The date the grant award was created. Format: YYYY-MM-DD or YYYY-MM-DD HH:MM:SS.SSS if time is required. Date/Time is (UTC timezone)                                                                                                          |
| AWARD_START           | TIMESTAMP      | Y        | The timestamp the grant award begins. Format: YYYY-MM-DD or YYYY-MM-DD HH:MM:SS.SSS if time is required. Date/Time is (UTC timezone)                                                                                                          |
| AWARD_END             | TIMESTAMP      | Y        | The timestamp the grant award ends. Format: YYYY-MM-DD or YYYY-MM-DD HH:MM:SS.SSS if time is required. Date/Time is (UTC timezone)                                                                                                            |
| PRIMARY_FUNDER_NAME   | TEXT/255       | N        | The Primary Funder Name (Funder of original source of funds). If not set, the Direct Funder will also be set as Primary Funder.                                                                                                              |
| PRIMARY_FUNDER_CODE   | TEXT/255       | N        | The Primary Funder unique identifier (institutional Funder ID). If not set, the Direct Funder will also be set as Primary Funder.                                                                                                            |
| DIRECT_FUNDER_NAME    | TEXT/255       | Y        | The Direct Funder Name (Funder from which funds are directly received)                                                                                                                                                                        |
| DIRECT_FUNDER_CODE    | TEXT/255       | Y        | The Direct Funder unique identifier (institutional Funder ID)                                                                                                                                                                                 |
| PI_FIRST_NAME         | TEXT/255       | Y        | First name of PI                                                                                                                                                                                                                             |
| PI_MIDDLE_NAME        | TEXT/255       | N        | Middle name of PI                                                                                                                                                                                                                            |
| PI_LAST_NAME          | TEXT/255       | Y        | Last name of PI                                                                                                                                                                                                                              |
| PI_EMAIL              | TEXT/255       | Y        | Email address of PI                                                                                                                                                                                                                          |
| PI_INSTITUTIONAL_ID   | TEXT/128       | Y        | Institutional User ID of PI. This is typically the User ID in the institution's Identity Access Management system. This value is optional if PI_EMPLOYEE_ID exists. If User ID exists in this record, it is important that the User ID is available to PASS during the authentication process. |
| PI_EMPLOYEE_ID        | TEXT/128       | Y        | Employee ID of PI. This value is optional if PI_INSTITUTIONAL_ID exists. If Employee ID exists in this record, it is important that the Employee ID is available to PASS during the authentication process.                                    |
| PI_ROLE               | TEXT/1         | Y        | Role of PI on grant (PI or Co-PI). Valid values: P, C. P=PI, C=Co-PI                                                                                                                                                                          |
| UPDATE_TIMESTAMP      | TIMESTAMP      | N        | Last update timestamp. Format: YYYY-MM-DD or YYYY-MM-DD HH:MM:SS.SSS if time is required. Date/Time is (UTC timezone)                                                                                                                         |

### Usage

Refer to the application.properties file to determine which properties that need runtime values set. The grant loader is a spring boot application, so use the standard Spring Boot configuration functionality according to the
[Spring Boot Configuration](https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features.external-config) documentation.

Here is an example using Java system properties `-D`. Change the version `1.8.0-SNAPSHOT` to the version of the `jhu-grant-loader` jar that is being used.
```shell
java -jar jhu-grant-loader-1.8.0-SNAPSHOT.jar -a load file:./grant-data.csv
```

#### Arguments

You can run the above command with `-h` to get a full list of arguments for the grant loader.

In this example below using the -a parameter instructs the grant loader to load data from a file CSV file. The action for the -a parameter is `pull` or `load`, to restrict the operation of the application to only pull data from Grant source system to store in a file, or to only load into PASS data taken from a stored file, respectively.

```shell
java -jar jhu-grant-loader-1.8.0-SNAPSHOT.jar -a load file:./grant-data.csv
```

In another example below, `startDateTime` and `awardEndDate` are used as parameters to limit the date range of the grant data. Since no action is specified, the default is to perform a pull followed directly by a load, using the default connection source.

```shell
java -jar jhu-grant-loader-1.8.0-SNAPSHOT.jar -startDateTime <yyyy-mm-dd hh:mm:ss.m{mm}> -awardEndDate <MM/dd/yyyy>
```

### Running the Grant Loader in Docker

#### Run PASS Docker

Since the Grant Loader will load data from a CSV into PASS, you will need an instance of PASS running. The quickest way to accomplish this is to run [PASS docker](../../welcome-guide/setup-run-pass.md).

Start `pass-docker` in local mode by running with the `dock-compose.yml` and `eclipse.pass.local.yml` configurations:

```shell
docker compose -f docker-compose.yml -f eclipse-pass.local.yml up -d --no-build --quiet-pull
```

Once pass-docker is up and the loader container is done running, open a browser and go to http://localhost:8080/ and login with nih-user. More details about this account can be found on the [PASS docker](../../welcome-guide/setup-run-pass.md) page. Go to Grants tab to view all the grants. This page will be empty, but after running the Grant Loader it will have all the grants from the CSV file provided.

#### Setup Grant Loader Test Directory

1. Create a directory named `grantloadertest`
2. Change to the directory: `cd grantloadertest`.
3. Create the following files:
   - `grant_update_timestamps` (empty)
   - `policy.properties` (empty)
   - `env.list` with content:
   ```text
   APP_HOME_ENV=/data/grantloader
   POLICY_PROP_PATH=file:/data/grantloader/policy.properties
   PASS_CLIENT_URL=http://localhost:8080
   PASS_CLIENT_USER=<value from .eclipse-pass.local_env in pass-docker PASS_CORE_BACKEND_USER>
   PASS_CLIENT_PASSWORD=<value from .eclipse-pass.local_env in pass-docker PASS_CORE_BACKEND_PASSWORD>
   ```
4. Copy your grant CSV file to the `grantloadertest` directory.
5. Open a new terminal and cd to the pass-docker directory.

#### Running Grant Loader Load

For testing purposes, we need to associate nih-user to a grant row in the CSV.

1. Modify one of your grant rows to change the user fields to:
```text
   Ser,,Nihu,nihuser@jhu.edu,NIHUSER,118110
```
2. Open a new terminal window and navigate to `grantloadertest`
3. Run 
```text
docker run -it -v ./grantloadertest:/data/grantloader --env-file ./grantloadertest/env.list --network host ghcr.io/eclipse-pass/jhu-grant-loader:1.8.0-SNAPSHOT -a load /data/grantloader/<your_file>.csv`
```
4. Once done, refresh the Grants tab in the browser to see your grant loaded.

Troubleshooting:
- If the grant csv contains new Funders, you should figure out PASS policy ID and put the funder local key to policy ID mapping in policy.properties (i.e. funder_local_key=pass_policy_id) before running the grant loader docker command.
- If running on Windows references to the current directory should use `${PWD}` for Powershell or `%cd%` for Windows Command Line. 

### Grant Loader Classes & Data Flow Overview

1. Initialization and Configuration:
    - The application initializes with `GrantLoaderCLI`, which sets up the `GrantLoaderApp` with configurations from `GrantLoaderConfig`.
    - Spring Boot profiles are used to load institution-specific configurations.
2. Data Retrieval:
    - `GrantLoaderApp` uses the `GrantConnector` interface to retrieve data from the data source (e.g., database, CSV file).
    - The `CoeusConnector` implementation (e.g., for JHU) fetches the data and returns it as a list of `GrantIngestRecord` objects.
3. Data Processing:
    - The `GrantIngestRecord` objects are built by the `CoeusConnecter` by the `retrieveUpdates` method.
      - The `CoeusConnecter` implements `GrantConnector`, and is specific the COEUS database at JHU. Another institution should have an implementing class for their institution.
    - A `LocalKey` is built using utility methods from `GrantDataUtils` and is used by the `AbstractDefaultPassUpdater`
4. Data Ingestion:
    - The processed grant data is passed to the `JhuPassUpdater`, which extends `AbstractDefaultPassUpdater`.
      - An institution with specific needs for updating their data should extend the `AbstractDefaultPassUpdater`. The `JhuPassUpdater` is specific to JHU implementation.
    - The `JhuPassUpdater` updates PASS objects (grants, users, funders) in the PASS repository, and interacts with the PassClient to perform the actual create and update operations.
5. Error Handling:
    - Exceptions specific to the data retrieval or ingestion are handled by `GrantDataException`.
    - Errors are logged, and appropriate messages are reported to the CLI user via `PassCliException`.
6. Statistics Tracking:
    - The `PassUpdateStatistics` class tracks the number of grants, funders, and users created or updated.
    - Statistics are updated in the `PassUpdater` and can be reset or reported.

## Next Step / Institution Configuration

Institutional configuration is going to be highly dependent on where the institutional grant data comes from. At JHU, we have a Postgres database and the data is pulled from the database using [AWS Batch and ECS](../../welcome-guide/deployment-architecture.md#pass-deployment-architecture). There can be multiple ways to set up the infrastructure, but the simplest setup is to have a CSV file exported to a directory where the Grant Loader can ingest the file using the `-a` parameter.