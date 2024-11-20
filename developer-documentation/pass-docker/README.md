# PASS Docker

Developer-focused PASS runtime, which provides the PASS project and all of its dependent services using docker-compose. PASS Docker provides Docker images that can be used for running PASS in different environments including a local test instance and production.

## Technologies Utilized
* [Docker](https://www.docker.com/get-started/)
* [Docker Compose](https://docs.docker.com/compose/)


## Local Test Environment
The `demo` yml file describes an early system meant to demonstrate new technologies and services that are available in PASS.

### Running A Local Test Instance of PASS
In order to run a local test instance of the PASS project using Docker Compose you need to specify the correct `yml` file and the correct `env` file.

#### Run Without Deposit Services
In order to run a local instance **_without_** deposit-service, ftp, and dspace, you can run the following command:
```
docker compose -f docker-compose.yml -f eclipse-pass.local.yml up -d --no-build --quiet-pull --pull always
```

#### Run With Deposit Services and DSpace
In order to run a local instance **_with_** deposit-service, ftp, dspace, you can run the following command:
```
docker compose -p pass-docker -f docker-compose.yml -f eclipse-pass.local.yml -f docker-compose-deposit.yml -f docker-compose-dspace.yml up -d --no-build --quiet-pull --pull always
```

##### Testing DSpace Integration with a Local Test Instance of PASS
Run the following to create a test admin user in dspace:
```
docker compose -p pass-docker -f dspace-cli.yml run --rm dspace-cli create-administrator -e test@test.edu -f admin -l user -p admin -c en
```

Run the following to load sample data into dspace:
```
docker compose -p pass-docker -f dspace-cli.yml -f dspace-cli.ingest.yml run --rm dspace-cli
```

#### Run With Deposit Services and InvenioRDM
In order to run a local instance **_with_** deposit-service, ftp, InvenioRDM, you can run the following command:

Refer to [PASS Docker Testing InvenioRDM](./invenio-rdm.md) for instructions managing a local test InvenioRDM instance that will communicate with `pass-docker`.

**Note this configuration for deposit services and InvenioRDM is for local testing only and is not intended for production.**

First, start the local test InvenioRDM and add the `Access token` to the appropriate `env` file by following the steps highlighted in [PASS Docker Testing InvenioRDM](./invenio-rdm.md). 

After the InvenioRDM service is up and running, run the following commands in the `pass-docker` directory:

```console
docker compose -p pass-docker -f docker-compose.yml -f eclipse-pass.local.yml -f docker-compose-deposit.yml -f docker-compose-deposit-invenio-rdm.yml up -d --no-build --quiet-pull --pull always
```

#### InvenioRDM Submission Requirements

When creating a Submission in PASS that will deposit into InvenioRDM with this local test configuration, there are a 
few requirements for Submission input:

* Open a browser and go to [http://localhost:8080/app/](http://localhost:8080/app/)
* Login using the staff1 user
* Create a new submission
* On the Grants step, select the `invenio-test-awd-num-1` grant
* On the Details step:
  * Enter a Publisher
  * Enter Publication Date (format: yyyy-mm-dd)
  * Enter Author in format <last_name>, <first_name>


### Stopping a Local Instance
In order to stop a local instance, you can run the following command:
```
docker compose -p pass-docker down -v
```
Note the `-v` to remove the volumes, **this is critical** so on subsequent starts, user data is not duplicated.


## Services

### idp

This service runs a Shibboleth Identity Provider using an image from [InCommon Trusted Access Platform Library](https://spaces.at.internet2.edu/display/ITAP/InCommon+Trusted+Access+Platform+Release).
Configuration files in the image are overridden on startup by using files in `idp/`. This service is intended for testing only.

#### Environment variables
* `IDP_HOST=http://localhost:9080`
* `SP_LOGIN=http://localhost:8080/login/saml2/sso/pass`

Separately there is a non-container environment variable `IDP_INTERNAL_PORT` which is used to set the internal port on the IDP container to which 9080 maps.
The default is 8080. This can be used to make 9080 support https by setting it to 4443 in the docker compose environment. One way to do this is by adding
`IDP_INTERNAL_PORT=4443` to the docker compose command. Note that `-e` should not be used because it is for container environment variables.


### ldap

This service runs the 389 Directory Server which is a LDAP server. It is used by the IDP as a source of information on users.
The users in ` ldap/pass.ldif` are loaded on startup.This service is intended for testing only.


### pass-core

* [Repository](https://github.com/eclipse-pass/pass-core)
* [Package](https://github.com/orgs/eclipse-pass/packages/container/package/pass-core-main)

Presents a JSON:API window to the backend from behind the authentication layer. Swagger is not currently implemented and as a result it is unreachable. This service provides data and web APIs to the application. This service supports SAML and HTTP basic authentication.

#### Environment variables

* `PASS_CORE_BASE_URL=http://localhost:8080` : Used when generating JSON API relationship links. Needs to be absolute and must change to match deployment environment
* `PASS_CORE_POSTGRES_PORT=5432`
* `PASS_CORE_BACKEND_USER=backend`
* `PASS_CORE_BACKEND_PASSWORD=backend`
* `PASS_CORE_APP_LOCATION=http://pass-ui:81/app/` : Resource location of pass ui resources
* `PASS_CORE_APP_CSP=default-src 'self';` : Content Security Policy header value
* `PASS_CORE_IDP_METADATA=http://idp:8080/idp/shibboleth` : Resource location of IDP metadata
* `PASS_CORE_SP_ID=https://sp.pass/shibboleth` : Identifier of pass-core as an SP
* `PASS_CORE_SP_KEY=file:///path/key`          : Resource location of SP key
* `PASS_CORE_SP_CERT=file:///path/cert`        : Resource location of SP certificate
* `PASS_CORE_LOGOUT_SUCCESS=/app/`             : Location user is redirected after logout
* `PASS_CORE_LOGOUT_DELETE_COOKIES="JSESSIONID /,shib_idp_session /idp"` : Cookies to delete on logout, "name path" separated by commas.
* `POSTGRES_USER=postgres`
* `POSTGRES_PASSWORD=postgres`
* `JDBC_DATABASE_URL=jdbc:postgresql://postgres:5432/pass`
* `JDBC_DATABASE_USERNAME=pass`
* `JDBC_DATABASE_PASSWORD=moo`
* `PASS_CORE_FILE_SERVICE_TYPE=S3`
* `PASS_CORE_S3_BUCKET_NAME=passcorefilestest`
* `PASS_CORE_S3_ENDPOINT=http://localstack:4566`


### postgres

A standard PostgreSQL database server with minimum modifications. This service's only interaction is with the [`pass-core`](https://github.com/eclipse-pass/pass-core) service.


### pass-ui

* [Repository](https://github.com/eclipse-pass/pass-ui)
* [Package](https://github.com/orgs/eclipse-pass/packages/container/package/pass-ui)

User interface for the PASS application. PASS-UI currently does not handle environment variables nicely, as a result environmental variables are baked into images at build time. The environment variables in the local test environment should not need to be adjusted between different deployment environments.

#### Environment variables
* `PASS_UI_PORT=81`
* `PASS_API_NAMESPACE=data`
* `PASS_UI_ROOT_URL=/app`
* `STATIC_CONFIG_URL=/app/config.json`
* `DOI_SERVICE_URL=/doiservice/journal`
* `MANUSCRIPT_SERVICE_LOOKUP_URL=/downloadservice/lookup`
* `MANUSCRIPT_SERVICE_DOWNLOAD_URL=/downloadservice/download`
* `POLICY_SERVICE_POLICY_ENDPOINT=/policyservice/policies`
* `POLICY_SERVICE_REPOSITORY_ENDPOINT=/policyservice/repositories`
* `SCHEMA_SERVICE_URL=/schemaservice`
* `USER_SERVICE_URL=/pass-user-service/whoami`


### loader

A lightweight Docker image that performs a `curl` command in order to bootstrap the environment with data from `demo_data.json`


## Running Acceptance Tests

* [Repository](https://github.com/eclipse-pass/pass-acceptance-testing)

There is a small set of end-to-end smoke tests that can run against this environment for some validation of changes. These tests run automatically for new PRs that are opened against `main`, but they can also be run locally. In order to do this, first clone the repository with the tests.

Once you have the repository cloned, wait for the Docker Compose environment to start up and initialize. Once the Docker Composer environment is initialized run the tests directly. Run the following command from within the `pass-acceptance-testing` directory:

``` sh
yarn            # Installs project dependencies
yarn run test   # Runs tests
```


## Related Documentation:
* [PASS Docker Testing InvenioRDM](./invenio-rdm.md)
