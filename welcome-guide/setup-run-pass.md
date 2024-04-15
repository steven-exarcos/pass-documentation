# Setup and Run PASS Locally

## Introduction

The purpose of this article is to guide you through the process of getting PASS up and running locally using [Docker](https://www.docker.com/). If you're new to Docker, take a look at the [Get Started with Docker Guide](https://www.docker.com/get-started/). There are many other ways to deploy and run PASS, but this article focuses on getting PASS up and running on a local machine for a preview of the application. If looking to deploy PASS into a production environment, please take a look at the [PASS Infrastructure](../infrastructure-documenation/) and [Developer Documentation.](../developer-documentation/) &#x20;

### Requirements

* Docker&#x20;
  * Docker Engine version 20.10.21 or higher
* A minimum of 16GB of system memory is required, as Docker uses 8.5GB of virtual memory when starting up all services.

## Run PASS Locally

### Simple Setup

Running PASS locally requires a few simple steps and no configuration, as defaults and demo data are already provided. If you do want to explore the configuration of PASS, the [configuration section](setup-run-pass.md#configuration) of this page details how to edit the environment file.&#x20;

The first step is to clone or [download ](https://github.com/eclipse-pass/pass-docker/archive/refs/heads/main.zip)the code from the PASS Docker repository:

{% code title="Cloning PASS Docker" %}
```
git clone https://github.com/eclipse-pass/pass-docker.git
```
{% endcode %}

Once you've either cloned PASS docker or extracted it from the ZIP file, then you will want to use a command line tool to navigate to the root directory of PASS docker.&#x20;

<pre data-title="Navigate to PASS Docker" data-full-width="false"><code><strong>cd C:\Users\username\IdeaProjects\pass-docker
</strong></code></pre>

Now that you're at the root directory of PASS Docker run the following command:&#x20;

{% code title="Run Docker Compose" overflow="wrap" %}
```
docker compose -f docker-compose.yml -f eclipse-pass.local.yml up -d --no-build --quiet-pull --pull always
```
{% endcode %}

After running `docker compose` you should see the following images being pulled and started running in a container.

{% code title="Docker Compose Output" %}
```
 [+] Running 12/12
 - Network pass-docker_front         Created
 - Network pass-docker_back          Created
 - Volume "pass-docker_db"           Created
 - Container ldap                    Started
 - Container pass-ui                 Started
 - Container auth                    Started
 - Container proxy                   Started
 - Container localstack              Started
 - Container pass-docker-postgres-1  Started
 - Container idp                     Started
 - Container pass-core               Healthy
 - Container loader                  Started
```
{% endcode %}

After the container is running, PASS is now running locally on your machine! All you need to do now is navigate to [http://localhost:8080](http://localhost:8080) using a web browser, such as Firefox, Google Chrome, or Safari. From there you will see a login screen and can login using these test accounts:

<table><thead><tr><th width="199">User name</th><th width="120">Password</th><th>Description</th></tr></thead><tbody><tr><td>nih-user</td><td>moo</td><td>User that has NIH grants, and submissions on their behalf waiting for approval.</td></tr><tr><td>staff1</td><td>moo</td><td>User that has one NIH Grant, and no submissions.</td></tr><tr><td>staff2</td><td>moo</td><td>User that doesn't have any grants or submissions.</td></tr></tbody></table>

If you need to restart docker and re-run the containers, use the following command:

{% code title="Docker Compose Down" %}
```
docker compose -p pass-docker down -v
```
{% endcode %}

This will shutdown docker and remove all data associated with the application. This is important because when running `docker compose up` it loads fake data into the database. If you recreate the containers without previously destroying the data, duplicates will be inserted into the database.

### Advanced Setup

The simple setup contains the workflow of creating a submission and simulating the deposit, but it doesn't actually go anywhere. If you want to test the full integration workflow with deposit services and submitting to DSpace or to NIHMS you can run the following command:

{% code title="PASS with Deposit Services" overflow="wrap" %}
```
docker compose -p pass-docker -f docker-compose.yml -f eclipse-pass.local.yml -f docker-compose-deposit.yml -f docker-compose-dspace.yml up -d --no-build --quiet-pull --pull always
```
{% endcode %}

Add an administrator and sample data into DSpace:

{% code title="Add Administrator" overflow="wrap" %}
```
docker compose -p pass-docker -f dspace-cli.yml run --rm dspace-cli create-administrator -e test@test.edu -f admin -l user -p admin -c en
```
{% endcode %}

{% code title="Add Sample Data" overflow="wrap" %}
```
docker compose -p pass-docker -f dspace-cli.yml -f dspace-cli.ingest.yml run --rm dspace-cli
```
{% endcode %}

With Deposit Services and DSpace running in the container, when you run through the submission process and submit a manuscript to JScholarship it will make a deposit into DSpace. To view a deposit in the locally running DSpace instance navigate to [http://localhost:4000](http://localhost:4000) in your web browser. Login using username:`test@test.edu` and password: `admin`.

If submitting a deposit to [NIHMS](https://www.nihms.nih.gov), you can view the simulated deposit locally in the `pmc-sftp-server` image that is part of `pass-docker`.

The first step is getting the name of the `pmc-sftp-server` container by running the `docker ps` command:

```
docker ps
```

It will output the following:

{% code title="Docker ps Output" %}
```
CONTAINER ID   IMAGE                                                       COMMAND                  CREATED          STATUS                    PORTS                                                                    NAMES
c32ca1ae2905   dspace/dspace-solr:dspace-7.6                               "/bin/bash -c 'init-…"   41 minutes ago   Up 40 minutes             0.0.0.0:8983->8983/tcp                                                   dspacesolr
fb2d2c0c46c7   ghcr.io/eclipse-pass/deposit-services-core:1.6.0-SNAPSHOT   "./entrypoint.sh"        41 minutes ago   Up 40 minutes                                                                                      pass-deposit-services
8394fa5b55da   ghcr.io/eclipse-pass/idp:1.6.0-SNAPSHOT                     "./entrypoint.sh"        41 minutes ago   Up 41 minutes             4443/tcp, 8443/tcp                                                       idp
65e71a8770d4   dspace/dspace:dspace-7.6-test                               "/bin/bash -c 'while…"   41 minutes ago   Up 40 minutes             8000/tcp, 8009/tcp, 0.0.0.0:8080->8080/tcp                               dspace
df73b4336089   ghcr.io/eclipse-pass/pass-core-main:1.6.0-SNAPSHOT          "./entrypoint.sh"        41 minutes ago   Up 40 minutes (healthy)                                                                            pass-core
015446cc79c1   ghcr.io/eclipse-pass/pass-auth:1.6.0-SNAPSHOT               "docker-entrypoint.s…"   41 minutes ago   Up 41 minutes             80/tcp, 443/tcp                                                          auth
0b7b9956da40   localstack/localstack:3.2.0                                 "docker-entrypoint.sh"   41 minutes ago   Up 40 minutes (healthy)   127.0.0.1:4510-4559->4510-4559/tcp, 127.0.0.1:4566->4566/tcp, 5678/tcp   localstack
2ab832b2ec61   postgres:14-alpine                                          "docker-entrypoint.s…"   41 minutes ago   Up 41 minutes             5432/tcp                                                                 pass-docker-postgres-1
1b9fb04ddb02   dspace/dspace-postgres-pgcrypto:dspace-7.6                  "docker-entrypoint.s…"   41 minutes ago   Up 41 minutes             0.0.0.0:5432->5432/tcp                                                   dspacedb
6e8230eb7814   ghcr.io/eclipse-pass/pass-ui:1.6.0-SNAPSHOT                 "/bin/entrypoint.sh"     41 minutes ago   Up 41 minutes             80/tcp                                                                   pass-ui
84b402f8255c   ghcr.io/eclipse-pass/demo-ldap:1.6.0-SNAPSHOT               "./entrypoint.sh"        41 minutes ago   Up 41 minutes             389/tcp                                                                  ldap
1510d03d1f08   dspace/dspace-angular:dspace-7.6                            "docker-entrypoint.s…"   41 minutes ago   Up 41 minutes             0.0.0.0:4000->4000/tcp, 0.0.0.0:9876->9876/tcp                           dspace-angular
24e1ee4ac934   atmoz/sftp                                                  "/entrypoint pmcsftp…"   41 minutes ago   Up 41 minutes             0.0.0.0:2222->22/tcp                                                     pmc-sftp-server
4253aa6924a0   ghcr.io/eclipse-pass/proxy:1.6.0-SNAPSHOT                   "entrypoint.sh"          41 minutes ago   Up 40 minutes             0.0.0.0:80->80/tcp, 0.0.0.0:443->443/tcp                                 proxy
```
{% endcode %}

Get the container ID that is associated with `atmoz/sftp.`Using the container ID enter the SFTP container.

{% code title="Get the Container ID" %}
```
docker ps -aqf name=pmc-sftp-server
```
{% endcode %}

Copy the result from the "Get Container ID" command. Then run the "Enter the SFTP Container" command by replacing the "PasteContainerIDHere" with the previously copied container ID.&#x20;

{% code title="Enter the SFTP Container" %}
```
docker exec -it PasteContainerIDHere /bin/bash
```
{% endcode %}

Now that you're in the SFTP server, navigate to the deposit. **Note:** the last directory that is a date will be the date when the deposit is made.

```
cd /home/pmcsftpuser/upload/2024-04-02
```

And you will see all the deposits that you've made to NIHMS:

```
root@24e1ee4ac934:/home/pmcsftpuser/upload/2024-04-02# ls
nihms-native-2022-05_2024-04-02_19-04-00_68.zip  nihms-native-2022-05_2024-04-02_19-04-48_68.zip  nihms-native-2022-05_2024-04-02_19-04-57_68.zip  nihms-native-2022-05_2024-04-02_19-04-58_68.zip
```

Congratulations! You've simulated a manuscript deposit to the institutional repository (JScholarship) and NIHMS!&#x20;

## Configuration

All the defaults in the docker env configuration files will run without any modification, but if you need to change a port, URLs, or other configurations you can do so using the `.eclipse-pass.local_env` file. It's recommend while testing to keep these values the defaults, but if you need to change a port for a specific reason you can do so by modifiying the `.eclipse-pass.local_env`environmental variable file.

## Troubleshooting

1. **Docker Compose Fails to Start the Services**

**Problem**: Users might encounter errors when running the `docker compose up` command due to various reasons such as network issues, Docker daemon not running, or insufficient permissions.

**Solution**: Ensure Docker is running on your machine. Check your internet connection and firewall settings. If Docker is running and no networking connections issues are present, then trying updating or installing the [latest version of Docker](https://www.docker.com/get-started/).

***

2. **Insufficient System Memory Error**

**Problem:** The application might fail or perform poorly if the system does not meet the minimum memory requirement.

**Solution:** Close unnecessary applications to free up memory. Consider increasing your system's memory if persistent issues occur. Ensure you have at least 16GB of system memory available as recommended. If you're running Docker on a Windows machine, ensure that WSL2 and Docker Compose V2 are enabled in the settings.

***

3. **Unable to Access PASS on the web browser**

**Problem:** After running the Docker compose command, the PASS application does not load or displays an error in the web browser.

**Solution:** Verify that the containers are running by running the `docker ps` command. When running without deposit services you should see the following containers running(**Note**: image version may differ as it will be updated in the future e.g. 1.6.0-SNAPSHOT):&#x20;

```
CONTAINER ID   IMAGE                                                COMMAND                  CREATED              STATUS                    PORTS                                                                    NAMES
813972bad937   ghcr.io/eclipse-pass/idp:1.6.0-SNAPSHOT              "./entrypoint.sh"        About a minute ago   Up 56 seconds             4443/tcp, 8443/tcp                                                       idp
2aafc2de282e   ghcr.io/eclipse-pass/pass-core-main:1.6.0-SNAPSHOT   "./entrypoint.sh"        About a minute ago   Up 43 seconds (healthy)                                                                            pass-core
960fbbed7458   ghcr.io/eclipse-pass/pass-auth:1.6.0-SNAPSHOT        "docker-entrypoint.s…"   About a minute ago   Up 57 seconds             80/tcp, 443/tcp                                                          auth
db263a61122d   localstack/localstack:3.2.0                          "docker-entrypoint.sh"   About a minute ago   Up 44 seconds (healthy)   127.0.0.1:4510-4559->4510-4559/tcp, 127.0.0.1:4566->4566/tcp, 5678/tcp   localstack
663871b8a92b   postgres:14-alpine                                   "docker-entrypoint.s…"   About a minute ago   Up 58 seconds             5432/tcp                                                                 pass-docker-postgres-1
b12319e6ccbb   ghcr.io/eclipse-pass/pass-ui:1.6.0-SNAPSHOT          "/bin/entrypoint.sh"     About a minute ago   Up 58 seconds             80/tcp                                                                   pass-ui
a51102277cff   ghcr.io/eclipse-pass/demo-ldap:1.6.0-SNAPSHOT        "./entrypoint.sh"        About a minute ago   Up 58 seconds             389/tcp                                                                  ldap
576ff2aae621   ghcr.io/eclipse-pass/proxy:1.6.0-SNAPSHOT            "entrypoint.sh"          About a minute ago   Up 54 seconds             0.0.0.0:80->80/tcp, 0.0.0.0:443->443/tcp                                 proxy
```

When running with Deposit-Services and DSpace, as specified in the [advanced setup](setup-run-pass.md#advanced-setup), you will see the following containers running:

```
CONTAINER ID   IMAGE                                                       COMMAND                  CREATED          STATUS                             PORTS                                                                    NAMES
8359f7400413   ghcr.io/eclipse-pass/idp:1.6.0-SNAPSHOT                     "./entrypoint.sh"        41 seconds ago   Up 33 seconds                      4443/tcp, 8443/tcp                                                       idp
0d0abd03c480   dspace/dspace-solr:dspace-7.6                               "/bin/bash -c 'init-…"   41 seconds ago   Up 26 seconds                      0.0.0.0:8983->8983/tcp                                                   dspacesolr
b14227cec259   ghcr.io/eclipse-pass/deposit-services-core:1.6.0-SNAPSHOT   "./entrypoint.sh"        41 seconds ago   Up 9 seconds                                                                                                pass-deposit-services
238a6183e1ab   ghcr.io/eclipse-pass/pass-core-main:1.6.0-SNAPSHOT          "./entrypoint.sh"        41 seconds ago   Up 12 seconds (health: starting)                                                                            pass-core
a7a24772c12d   dspace/dspace:dspace-7.6-test                               "/bin/bash -c 'while…"   41 seconds ago   Up 29 seconds                      8000/tcp, 8009/tcp, 0.0.0.0:8080->8080/tcp                               dspace
bc3ea5a2ec20   ghcr.io/eclipse-pass/pass-auth:1.6.0-SNAPSHOT               "docker-entrypoint.s…"   43 seconds ago   Up 35 seconds                      80/tcp, 443/tcp                                                          auth
a7403c2b34a6   localstack/localstack:3.2.0                                 "docker-entrypoint.sh"   43 seconds ago   Up 15 seconds (healthy)            127.0.0.1:4510-4559->4510-4559/tcp, 127.0.0.1:4566->4566/tcp, 5678/tcp   localstack
135dc909b2c1   postgres:14-alpine                                          "docker-entrypoint.s…"   43 seconds ago   Up 35 seconds                      5432/tcp                                                                 pass-docker-postgres-1
f8c24a38d438   ghcr.io/eclipse-pass/proxy:1.6.0-SNAPSHOT                   "entrypoint.sh"          43 seconds ago   Up 29 seconds                      0.0.0.0:80->80/tcp, 0.0.0.0:443->443/tcp                                 proxy
d9bb301597f4   ghcr.io/eclipse-pass/demo-ldap:1.6.0-SNAPSHOT               "./entrypoint.sh"        43 seconds ago   Up 37 seconds                      389/tcp                                                                  ldap
340435bf059f   ghcr.io/eclipse-pass/pass-ui:1.6.0-SNAPSHOT                 "/bin/entrypoint.sh"     43 seconds ago   Up 35 seconds                      80/tcp                                                                   pass-ui
e6137acc145a   dspace/dspace-angular:dspace-7.6                            "docker-entrypoint.s…"   43 seconds ago   Up 32 seconds                      0.0.0.0:4000->4000/tcp, 0.0.0.0:9876->9876/tcp                           dspace-angular
778c5cec16d7   atmoz/sftp                                                  "/entrypoint pmcsftp…"   43 seconds ago   Up 33 seconds                      0.0.0.0:2222->22/tcp                                                     pmc-sftp-server
66daca298f22   dspace/dspace-postgres-pgcrypto:dspace-7.6                  "docker-entrypoint.s…"   43 seconds ago   Up 34 seconds                      0.0.0.0:5432->5432/tcp                                                   dspacedb
```

If some of the containers are not running try `docker compose -p pass-docker down -v` and restart the containers by running the docker compose command mentioned in the [simple ](setup-run-pass.md#simple-setup)or[ advanced setup](setup-run-pass.md#advanced-setup) sections.
