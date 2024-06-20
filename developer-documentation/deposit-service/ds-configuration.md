# Deposit Services - Configuration

The primary mechanism for configuring Deposit Services is through environment variables. This aligns with the patterns
used in development and production infrastructure which rely on Docker and its approach to runtime configuration.

Secondary configuration is provided by Spring Boot application.properties. This configuration includes lower-level
parameters such as message queues, the base url of Pass-Core, etc.

## Production Configuration Variables

| Environment Variable                    | Default Value                | Description                                                                                                                                                                                                                      |
|-----------------------------------------|------------------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `DSPACE_HOST`                           | localhost                    | the IP address or host name of the server running the SWORD protocol version 2 endpoint                                                                                                                                          
| `DSPACE_PORT`                           | 8181                         | the TCP port exposing the SWORD protocol version 2 endpoint                                                                                                                                                                      
| `DSPACE_SERVER`                         | dspace                       | the domain name of the DSpace server                                                                                                                                                                      
| `PMC_FTP_HOST`                          | localhost                    | the IP address or  host name of the NIH FTP server                                                                                                                                                                               
| `PMC_FTP_PORT`                          | 21                           | the TCP control port of the NIH FTP server                                                                                                                                                                                       
| `PMC_FTP_USER`                          | nihmsftpuser                 | the PMC S/FTP user                                                                                                                                                                                       
| `PMC_FTP_PASSWORD`                      | nihmsftppass                 | the PMC S/FTP password                                                                                                                                                                                     
| `PASS_DEPOSIT_QUEUE_SUBMISSION_NAME`    | submission                   | the name of the JMS queue that has messages pertaining to `Submission` resources                                                                                                        
| `PASS_DEPOSIT_QUEUE_DEPOSIT_NAME`       | deposit                      | the name of the JMS queue that has messages pertaining to `Deposit` resources                                                                                                            
| `PASS_DEPOSIT_REPOSITORY_CONFIGURATION` | classpath:/repositories.json | points to a json file containing the configuration for the transport of custodial content to remote repositories. Values must be [Spring Resource URIs][1]. See below for customizing the repository configuration values. 
| `PASS_CLIENT_URL`                       | localhost:8080               | the URL used to communicate with the PASS Core API. Normally this variable does not need to be changed (see note below)                                                                                                          
| `PASS_CLIENT_PASSWORD`                  | fakepassword                 | the password used for `Basic` HTTP authentication to the PASS Core API                                                                                                                                                           
| `PASS_CLIENT_USER`                      | fakeuser                     | the username used for `Basic` HTTP authentication to the PASS Core API                                                                                                                                                           

## Repositories Configuration

The Repository configuration contains the parameters used for connecting and depositing custodial material to downstream
repositories. The format of the configuration file is JSON, defining multiple downstream repositories in a single file.

Each repository configuration has a top-level key that is used to identify a particular configuration. Importantly, each
top-level key _must_ map to a [`Repository` resource][5] within the PASS repository. This implies that the top-level
keys in `repositories.json` are not arbitrary. In fact, the top level key must be:

* the value of a `Repository.repositoryKey` field (of a `Repository` resource in the PASS repository)

Deposit Services comes with a default repository configuration, but a production environment will want to override the
default. Defaults are overridden by creating a copy of the default configuration, editing it to suit, and
setting `PASS_DEPOSIT_REPOSITORY_CONFIGURATION` to point to the new location.

Deposit Services is configured at runtime by a configuration file referenced by the system
property `pass.deposit.repository.configuration` or the environment variable `PASS_DEPOSIT_REPOSITORY_CONFIGURATION`. By
default (i.e. if the system property or environment variable are not present) the classpath
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

#### Customizing Repository Configuration Elements

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