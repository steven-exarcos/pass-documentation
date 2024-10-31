# Deposit Services - Configuration

The primary mechanism for configuring Deposit Services is through environment variables. This aligns with the patterns
used in development and production infrastructure which rely on Docker and its approach to runtime configuration.

Secondary configuration is provided by Spring Boot `application.properties`. This configuration includes lower-level
parameters such as message queues, the base URL of Pass Core, etc.

## Production Configuration Variables

| Environment Variable                     | Default Value                | Description                                                                                                                                                 |
|------------------------------------------|------------------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `DSPACE_HOST`                            |                              | IP address or host name of the server running the SWORD protocol version 2 endpoint                                                                         |
| `DSPACE_PORT`                            |                              | TCP port exposing the SWORD protocol version 2 endpoint                                                                                                     |
| `DSPACE_SERVER`                          |                              | Domain name of the DSpace server                                                                                                                            |
| `DSPACE_USER`                            |                              | DSpace user                                                                                                                                                 |
| `DSPACE_PASSWORD`                        |                              | DSpace password                                                                                                                                             |
| `DSPACE_API_PROTOCOL`                    |                              | Dspace API protocol                                                                                                                                         |
| `DSPACE_API_PATH`                        |                              | Dspace API path                                                                                                                                             |
| `INVENIORDM_API_BASE_URL`                |                              | Base URL for InvenioRDM API                                                                                                                                 |
| `INVENIORDM_VERIFY_SSL_CERT`             | true                         | Required since the localhost InvenioRDM runs with a self-signed certificate                                                                                 |
| `INVENIORDM_API_TOKEN`                   |                              | InvenioRDM API token                                                                                                                                        |
| `PMC_FTP_HOST`                           |                              | IP address or host name of the NIH FTP server                                                                                                               |
| `PMC_FTP_PORT`                           |                              | TCP control port of the NIH FTP server                                                                                                                      |
| `PMC_FTP_USER`                           |                              | PMC S/FTP user                                                                                                                                              |
| `PMC_FTP_PASSWORD`                       |                              | PMC S/FTP password                                                                                                                                          |
| `PASS_DEPOSIT_QUEUE_SUBMISSION_NAME`     | submission                   | Name of the JMS queue that has messages pertaining to `Submission` resources                                                                                |
| `PASS_DEPOSIT_QUEUE_DEPOSIT_NAME`        | deposit                      | Name of the JMS queue that has messages pertaining to `Deposit` resources                                                                                   |
| `PASS_DEPOSIT_REPOSITORY_CONFIGURATION`  | classpath:/repositories.json | Points to a json file containing the configuration for the transport of custodial content to remote repositories. Values must be [Spring Resource URIs][1]. |
| `PASS_CORE_URL`                          |                              | URL used to communicate with the PASS Core API. Normally this variable does not need to be changed.                                                         |
| `PASS_CORE_PASSWORD`                     |                              | Password used for `Basic` HTTP authentication to the PASS Core API                                                                                          |
| `PASS_CORE_USER`                         |                              | Username used for `Basic` HTTP authentication to the PASS Core API                                                                                          |
| `NIHMS_MAIL_HOST`                        |                              | Host URL of the email service that will receive NIHMS emails regarding deposit statuses.                                                                    |
| `NIHMS_MAIL_PORT`                        |                              | Port of the email service that that will receive NIHMS emails regarding deposit statuses.                                                                   |
| `NIHMS_MAIL_USERNAME`                    |                              | Email address that will receive the NIHMS emails regarding deposit statuses.                                                                                |
| `NIHMS_MAIL_PASSWORD`                    |                              | Password of the email address that will receive the NIHMS emails regarding deposit statuses.                                                                |
| `NIHMS_MAIL_TENANT_ID`                   |                              | The tenant ID if the `NIHMS_MAIL_HOST` is a cloud provided email services (e.g. Office 365).                                                                |
| `NIHMS_MAIL_CLIENT_ID`                   |                              | The client ID if the `NIHMS_MAIL_HOST` is a cloud provided email services (e.g. Office 365).                                                                |
| `NIHMS_MAIL_CLIENT_SECRET`               |                              | The client secret if the `NIHMS_MAIL_HOST` is a cloud provided email services (e.g. Office 365).                                                            |
| `NIHMS_MAIL_AUTH`                        |                              | The type of authentication. Valid values are: `MS_EXCHANGE_OAUTH2` and `LOGIN`.                                                                             |
| `PASS_DEPOSIT_NIHMS_EMAIL_FROM`          |                              | The official email address that sends the error messages.                                                                                                   |                                                                        |
| `TEST_DATA_POLICY_TITLE`                 |                              | The title of the Policy to associate to the Deployment Test Funder of the Test Grant.                                                                       |
| `TEST_DATA_USER_EMAIL`                   |                              | The email of the User to set as the PI on the Test Grant.                                                                                                   |
| `TEST_DATA_SKIP_DEPOSITS`                | true                         | Whether to skip sending the Deployment Test Deposit to the remote repository or not.                                                                        |
| `TEST_DATA_DSPACE_REPO_KEY`              |                              | The repository key of the DSpace repository, if exists, that will be used to delete Deployment Test Deposit Items if made.                                  |

## Repositories Configuration

The Repository configuration contains the parameters used for connecting and depositing custodial material into 
downstream repositories. The format of the configuration file is JSON, defining multiple downstream repositories in a 
single file.

Each repository configuration has a top-level key that is used to identify a particular configuration. Importantly, each
top-level key _must_ map to a `Repository` resource within the PASS repository. This means that the top-level
keys in `repositories.json` are not arbitrary. In fact, the top level key must be:

* the value of a `Repository.repositoryKey` field, which is a `Repository` resource in the PASS repository.

Deposit Services comes with a default repository configuration, but a production environment should override the
default. Defaults are overridden by creating a copy of the default configuration, editing it to suit, and
setting `PASS_DEPOSIT_REPOSITORY_CONFIGURATION` to point to the new location.

Deposit Services is configured at runtime by a configuration file referenced by the system
property `pass.deposit.repository.configuration` or the environment variable `PASS_DEPOSIT_REPOSITORY_CONFIGURATION`. By
default, if the system property or environment variable are not present, the classpath
resource `/repositories.json` is used.

> Acceptable values for `PASS_DEPOSIT_REPOSITORY_CONFIGURATION` must be a form of [Spring Resource URI][1].

A possible repository configuration is replicated below:

```json
{
  "JScholarship": {
    "deposit-config": {
      "processing": {
        "beanName": "org.eclipse.pass.deposit.messaging.status.DefaultDepositStatusProcessor"
      },
      "mapping": {
        "http://dspace.org/state/archived": "accepted",
        "http://dspace.org/state/withdrawn": "rejected",
        "default-mapping": "submitted"
      }
    },
    "assembler": {
      "specification": "http://purl.org/net/sword/package/METSDSpaceSIP"
    },
    "transport-config": {
      "auth-realms": [
        {
          "mech": "basic",
          "username": "fakeuser",
          "password": "fakepass",
          "url": "https://jscholarship.library.jhu.edu/"
        }
      ],
      "protocol-binding": {
        "protocol": "SWORDv2",
        "username": "${dspace.user}",
        "password": "${dspace.password}",
        "server-fqdn": null,
        "server-port": null,
        "service-doc": "https://${dspace.server}/server/swordv2/servicedocument",
        "default-collection": "https://${dspace.server}/server/swordv2/collection/123456789/2",
        "on-behalf-of": null,
        "deposit-receipt": true,
        "user-agent": "pass-deposit/x.y.z"
      }
    }
  },
  "PubMed Central": {
    "deposit-config": {
      "processing": {
      },
      "mapping": {
        "INFO": "accepted",
        "ERROR": "rejected",
        "WARN": "rejected",
        "default-mapping": "submitted"
      }
    },
    "assembler": {
      "specification": "nihms-native-2017-07"
    },
    "transport-config": {
      "protocol-binding": {
        "protocol": "sftp",
        "username": "${pmc.ftp.user}",
        "password": "${pmc.ftp.password}",
        "server-fqdn": "${pmc.ftp.host}",
        "server-port": "${pmc.ftp.port}",
        "default-directory": "/upload/%s"
      }
    }
  }
}
```
### Customizing Repository Configuration Elements

The repository configuration above will not be suitable for production. A production deployment needs to provide
updated authentication credentials and ensure the correct value for the default SWORD collection URL
- `default-collection`. Each `transport-config` section should be reviewed for correctness, paying special attention
  to `protocol-binding` and `auth-realm` blocks: update `username` and `password` elements, and ensure correct values
  for URLs.

Values may be parameterized by any property or environment variable.

To create your own configuration, copy and paste the default configuration into an empty file and modify the JSON as
described above. The configuration _must_ be referenced by the `pass.deposit.repository.configuration` property, or is
environment equivalent `PASS_DEPOSIT_REPOSITORY_CONFIGURATION`. Allowed values are any [Spring Resource path][1] (
e.g. `classpath:/`, `classpath*:`, `file:`, `http://`, `https://`). For example, if your configuration is stored as a
file in `/etc/deposit-services.json`, then you would set the environment
variable `PASS_DEPOSIT_REPOSITORY_CONFIGURATION=file:/etc/deposit-services.json` prior to starting Deposit Services.
Likewise, if you kept the configuration accessible at a URL, you could
use `PASS_DEPOSIT_REPOSITORY_CONFIGURATION=http://example.org/deposit-services.json`.


## Configuring the NIHMS Email Service

The NIHMS email service is responsible for parsing emails from NIHMS that contain different error codes on deposits. 
Currently, NIHMS only communicates its deposit status via email. Setting up an email account that will receive emails
from NIHMS is a prerequisite. The email account that is set up should be represented by the value of 
`NIHMS_MAIL_USERNAME`. 

There are two authentication protocols supported by this service. A regular SMTP login credentials and Microsoft 
Exchange OAuth2. By providing a value of `LOGIN` or `MS_EXCHANGE_OAUTH2` for the environmental value, it will determine
which type of authentication to use. If using the `MS_EXCHANGE_OAUTH2` authentication type, then `NIHMS_MAIL_TENANT_ID`,
`NIHMS_MAIL_CLIENT_ID`, and `NIHMS_MAIL_CLIENT_SECRET` are required as well.

## Configuring the Deployment Test Data Service

The Deployment Test Data Service is responsible for cleaning up Submission/Deposit data created by Deployment Tests that
run against live PASS environments. To enable the job that executes the Deployment Test Data Service,
`pass.test.data.job.enabled` needs to be set to `true`. The `TEST_DATA*` environment variables need to be configured 
as well.

By default, Submissions created by Deployment Tests that are received by Deposit Services are fully processed; however,
the Deposit is not actually sent to the remote repository. This can be changed by setting `TEST_DATA_SKIP_DEPOSITS` to 
`false`, which will then send the Deposit to the remote repository. Additionally, when `TEST_DATA_SKIP_DEPOSITS`
is `false`, if the remote repository is a DSpace repository, the Deposit Item in DSpace will be deleted by the 
Deployment Test Data Service.

[1]: https://docs.spring.io/spring-framework/reference/core/resources.html#resources-implementations