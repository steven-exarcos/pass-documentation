# PASS Grant Loader

The PASS Grant Loader ingests grant data from an institution and maps the data to the appropriate PASS Objects in the PASS Data Model.

## Summary

This module comprises code for retrieving grant data from some kind of data source, and using that data to update
the PASS backend. Typically, the data pull will populate a data structure, which will then be consumed by a loader
class. While this sounds simple in theory, there are several considerations which may add to the complexity of
implementations. An implementor must first determine data requirements for the use of the data once it has been ingested
into PASS, and then map these requirements to data available from the data source. It may be that additional data from
other services may be needed in order to populate the data structures to be loaded into PASS. On the loading side, the
implementor may need to support different modes of ingesting data. Additional logic may be needed in the data loading
apparatus to resolve the fields in the data assembled by the pull process. For example, we will need to consider that
several systems may be updating PASS objects, and that other services may be more authoritative for certain fields than
the service providing the grant data. The JHU implementation is complex regarding these issues.

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

The grant loader uses Spring Boot Profiles to select the appropriate classes to be used for a given institution.
There is a property in application.properties named `spring.profiles.active` that needs to be set when starting the
grant loader.  This property can be set at runtime as well using the normal spring boot configuration functionality.
[Spring Boot Configuration](https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features.external-config)

The code has been factored to ease development for multiple institutions. These are the institution-specific classes
which typically need to be implemented and annotated with the `@Profile` annotation:

#### Connector

The Connector class connects to the data store for an institution's implementation, and operates on the data to supply,
in as standard a form as possible, the data to be consumed by the Updater.

#### Updater

The Updater class takes the data supplied by the Connector and creates or updates the corresponding objects in the PASS
repository accordingly. There is a Default class whose children may override certain substantive methods if the local
policies require.

### Profiles

#### JHU (jhu)

The JHU implementation is used to pull data from the COEUS/FIBI Oracle database views for the purpose of performing regular
updates. We look at grants which have been updated since a particular time (typically the time of the previous update),
join this with user and funder information associated with the grant, and then use this information to update the data
in the PASS backend. The JHU implementation also treats the COEUS/FIBI database as authoritative for all fields in the
data. If a grant is being passed in for update, it is assumed that all records for that grant are included in the
input.

## Grant Data

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

## Usage

Refer to the application.properties file to determine which properties that need runtime values set. The grant loader
is a spring boot application, so use the standard Spring Boot configuration functionality
[Spring Boot Configuration](https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features.external-config)

Here is an example using Java system properties `-D`.
```
    java -jar jhu-grant-loader-0.6.0-SNAPSHOT.jar -a load file:./grant-data.csv
```

### Arguments

You can run the above command with `-h` to get a full list of arguments for the grant loader.  In the example
above, we use `startDateTime` and `awardEndDate` as 

### Running the Grant Loader in Docker

#### Run PASS Docker

Since the Grant Loader will load data from a CSV into PASS, you will need an instance of PASS running. The quickest way to accomplish this is to run [PASS docker](../welcome-guide/setup-run-pass.md).

Start pass-docker in local mode by running: 

```shell
docker compose -f docker-compose.yml -f eclipse-pass.local.yml up -d --no-build --quiet-pull
```

Once pass-docker is up and the loader container is done running, open a browser and go to http://localhost:8080/ and login with nih-user. This account is a test user account created when starting pass-docker locally. Ask pass dev for password. Go to Grants tab to view all the grants. For right now this page will be empty, but after running the Grant Loader it should have all the grants from the CSV file provided.

#### Setup Grant Loader Test Directory

- Create directory named grantloadertest
- cd grantloadertest
- Create empty file named grant_update_timestamps
- Create empty file named policy.properties
- Create file named env.list in containing the following below:

```text
APP_HOME_ENV=/data/grantloader
POLICY_PROP_PATH=file:/data/grantloader/policy.properties
PASS_CLIENT_URL=http://localhost:8080
PASS_CLIENT_USER= (value from .eclipse-pass.local_env in pass-docker PASS_CORE_BACKEND_USER)
PASS_CLIENT_PASSWORD= (value from .eclipse-pass.local_env in pass-docker PASS_CORE_BACKEND_PASSWORD)
```

Copy your grant CSV file to grantloadertest dir


    Open a new terminal and cd to the pass-docker directory. You can checkout pass-docker from github here: https://github.com/eclipse-pass/pass-docker
    

Running Grant Loader Load

    For testing purposes, we need to associate nih-user to a grant row in the CSV. You can modify one of your grant rows to change the user fields to Ser,,Nihu,nihuser@jhu.edu,NIHUSER,118110
    Open a new terminal
    cd to dir *above the grantloadertest dir.
    Execute docker run -it -v ./grantloadertest:/data/grantloader --env-file ./grantloadertest/env.list --network host ghcr.io/eclipse-pass/jhu-grant-loader:1.6.0-SNAPSHOT -a load /data/grantloader/<your_file>.csv

Once done, refresh the Grants tab in the browser to see your grant loaded.

Note:
If the grant csv contains new Funders, you should figure out PASS policy ID and put the funder local key to policy ID mapping in policy.properties (i.e. funder_local_key=pass_policy_id) before running the grant loader docker command.


## Next Step / Institution Configuration

## Related Information