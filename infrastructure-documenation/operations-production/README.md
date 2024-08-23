# Operations/Production

The operations and production workflows, architecture, and infrastructure are composed of a variety of technologies,
layers, and techniques. The main important concept and understanding of this guide, is that PASS is designed to be 
platform independent, with a few exceptions. This guide will describe and summarize the way JHU decided to deploy the 
application, but this does mean it can't be done another way. One way of deploying PASS is using Amazon Web Services
(AWS) cloud computing services. By using the scalable infrastructure we are able to quickly adapt to different demands 
of usage. By using AWS, it opens up the ability to use infrastructure as code which gives more efficiencies by enabling
reuse of infrastructure between environments and aids in the CI/CD pipeline, providing consistent and quick deployments.
In addition to deploying PASS, other operation activities will be described such as monitoring, harvesting and loading 
data, and other communication between different services.

If you haven't already, a quick review of the Welcome Guide PASS Architecture article will provide a good foundation for
understanding the operations and production environment of PASS.

In this guide we step through various topics on JHU Operations and Production:

- [Knowledge Needed / Skills Inventory](./ops-know-need.md)
- [Technologies Utilized](./ops-tech-util.md)
- Technical Deep Dive
  - [PASS Design](./ops-design.md)
  - [How to Deploy](./ops-status.md)
  - [Monitoring](./ops-monitor.md)
  - [Assemblers](./ds-assemblers.md)
  - [Configuration](./ds-configuration.md)
  - [Amazon Web Services (AWS) Architecture](./ops-aws-arch.md)
- [Next Steps / Institution Configuration](./ds-new-institution.md)



## Technical Deep Dive

For each of the documents mentioned, please explain or give an example of the kind of content that should go in it.

- design document, (diagram of pass) + 
- how to deploy + 
- How to build a new environment -
- Monitoring +
- Developer access to services / logs (ssh tunneling, ops server, etc) - 
- Loader and Harvester jobs (COEUS, NIHMS ETL) + 
- Overview of amazon architecture, and role of each component, sample traffic flow +
    - e.g. Ember app communicating with public services via the proxy +
    - e.g. operations tasks that may occur on the backend/private network + 

### PASS Data & Backups

Placeholder doc to summarize data in PASS.
What are the data?
Where (in AWS) are they stored?
How are they protected / backed-up?

### PASS Release Versioning

## TODO edit this so it reflects the current Release versioning scheme

A release of PASS is made up of Java artifacts, Node modules, and Docker images. Maven builds all the Java artifacts and
many of the Docker images. The Node based pass-ember adapter is released to the NPM registry. The Node based pass-ui is
released as a Docker image in pass-docker. The Maven releases are the most complicated part of the overall process. 

In this approach all the maven modules are composed together into a single hierarchy such that they can all be
processed by the Maven reactor and the Maven release plugin used. This allows all the Java modules to be released with a
single command. Note that for every project with a different version, the development and release versions must be
passed as arguments or provided interactively. Also the Maven reactor won’t be able to figure out the dependencies
introduced by integration test Docker images unless they are specified in the project poms.

There are a few approaches that might be used to assemble this super module. The main pass repository which hosts the
root PASS pom could have all our Maven repositories listed as Maven submodules by adding each corresponding repository
as a Git submodule. Some investigation makes it look like this should work. The release plugin needs to get configured
to handle the submodules when it does a checkout. But more generally this seems a little clunky and complicated. A lot
gets linked into our main repository.

Another possibility is writing a script which assembles all the Maven repositories to be released into a hierarchy with
the root PASS pom at the top and edit the root pom to have them as submodules. But this won’t work because the code to
be released must be checked into a repository. We can workaround that problem by creating our own local Github repo and
doing the release against that. The local repository would have exports of every other PASS maven repository main
branch. That leaves a final problem of getting the changes made to the local repository (version changes and tags) into
the corresponding upstream repositories. This seems awkward, but doable.
Release separately

Instead of assembling a super pom and using the Maven reactor to handle dependencies between PASS projects, a script
could just release each project separately. This must be done in dependency order which would be hard coded into the
script. The dependency order is not hard to figure out. See above for an attempt. If every PASS project is on a
different version, the script would have to manage all of those versions. In addition given, that we would be going
through a list of modules in a certain order, adding support for resumption would not be difficult. Overall this seems
like the simplest approach.

Decisions and recommendations
The team has decided to use a single version across all of PASS. This has a number of benefits, not the least of which
is ease of understanding. Java projects and Docker images should be moved to the new version which is 0.1.0-SNAPSHOT.

I recommend that the Maven release process does not use a super module and instead releases projects separately in
dependency order.

### How JHU Uses AWS SNS Notifications and Cloudwatch for Monitoring

#### Assumptions

- PASS running AWS using RDS for Postgres that is not publicly accessible.
- Institution is using AWS SSO to authenticate.


### NIHMS API Token Generation

eRA Commons Login: https://public.era.nih.gov/commonsplus/public/login.era

API Token Generation*: PMC
*select using eRA commons to login

System Owner: The system owner is the NIH, but account management is delegated to a University’s Office of Sponsored
Research. In JHU’s case, this is: Johns Hopkins University Research Administration (JHURA). For any other university
setting up their own Nihms-data loader, it will be the Office of Sponsored Research that creates the account for the
API key.

Account Setup: In the case of JHU, the account needs to be set up by JHU Research Administration (JHURA). They will need
to have the following permissions in order to create an account: SO, AO, AA, or BO and they cannot be affiliated with
more than one institution.

If any modifications to the account need to be made, such as changing the associated email with the account, it will
need to be done via the eRA administrator.

Association with Pass: This account is used by the Nihms-loader in Pass-Support. It is responsible for generating an
API token, with a lifespan of 3 months, used to pull PACM data.

Other Documentation:
https://www.ncbi.nlm.nih.gov/pmc/utils/pacm/static/pacm-user-guide.pdf

### Eclipse Ops

The following site are managed by the Eclipse Foundation.
URL
Notes
Status
eclipse-pass.org
Github Pages HTML / CSS
running-locally (DNS not configured)
demo.eclipse-pass.org
GitHub Self-Hosted Runner / Docker Compose
running

The underlying infrastructure for each is documented here
Infra
User
Details
Github Pages HTML / CSS
N/A
Automatic via GitHub Actions
GitHub Self-Hosted Runner / Docker Compose
githubrunner
Run from /opt/githubrunner/pass-docker/pass-docker/pass-docker

If you are looking for more detailed developer-oriented instructions to help debug an issue with the above, please refer
to managing our demo servers documentation.
Configurations
The .eclipsefdn project allows our team to self-service several aspect of our organization via a tool called Otterdog
and secrets in Bitwarden.
These .eclipsefdn repo / tools gives access to:
Organization Settings
Organization Webhooks
Repositories and their settings
Branch Protection Rules
Learn more about Otterdog here.
Deployment
eclipse-pass.org
This is managed with GitHub Pages from the eclipse-pass.github.io repo.
Sync Demo To Marketing
The demo.eclipse-pass.org site includes marketing information about PASS that we currently synchronize with the main
eclipse-pass.org site.
Download the publicly accessible site.
This can be done with a tool like SiteSucker on a Mac.

Update the eclipse-pass.github.io repo.
In a new branch, copy all files from the download site.
Fix references to the demo site
Any link to "idp" for login purposes, such as
<a href="idp/..." class="btn btn-primary my-3 pl-4 pr-4 ember-view">Use the Demo</a>
Should be updated to
<a href="https://demo.eclipse-pass.org/login/jhu" class="btn btn-primary my-3 pl-4 pr-4 ember-view">Use the Demo</a>
PR branch into main
Once the code is in main, the site will automatically be deployed to eclipse-pass.org.

demo.eclipse-pass.org
The demo.eclipse-pass.org site is deployed on demand using GitHub actions. Note that this site is not yet publically
available.
The demo system is based on the pass-docker project and you can view the available actions including the Deploy passdemo
action
Publish Via GitHub.com
To publish an update, access the actions page.

You can then run the workflow

And watch the deploy.

Locally Run Via Docker Compose
The demo.eclipse-pass.org site is managed using Docker Compose.
To run locally first get a local copy of pass-docker then run it with the following commands
cd pass-docker && \
docker-compose -f eclipse-pass.base.yml -f eclipse-pass.demo.yml pull && \
docker-compose -f eclipse-pass.base.yml -f eclipse-pass.demo.yml up

For more information debugging the deployment to our demo servers please refer to our demo deploy documentation.
ightly.eclipse-pass.org) please refer to our demo deploy documentation.
Infrastructure
A bootstrapped server installer to run a stand-alone PASS application via the pass-docker project
Which will do the following
Install docker
Install github self-hosted runners
Intall pass-docker
Configure pass docker for a GH runner
To configure the self-hosted runner you will need the GITHUB token, replacing the XXX with the actual token value.
GITHUB_RUNNER_TOKEN=XXX

Learn more from the installer script itself.
Run PASS
To manually run the application, execute
cd /src/pass-docker && \
sudo docker-compose pull && \
sudo docker-compose up
If you want to run in the background then use -d.
sudo docker-compose up -d
Troubleshooting
Access Eclipse Servers
To access the Eclipse infrastructure you will need a bastion.eclipse.org login. If you do not have a bastion login, then
you will need to reach out to a team member for instructions (and permission) to get such a login.
Server
IP

demo.eclipse-pass.org
ssh ${ME}@bastion.eclipse.org -t ssh ${ME}@172.30.206.15

You will need to change ME to your username
ME=myusername
Once on the server, then CD into the directory and change to the githubrunner user
cd /opt/githubrunner/pass-docker/pass-docker/pass-docker
sudo su githubrunner
Now you have full access to the application and can run the application as shown below
Run Remote Apps
From our GitHub actions you will see the specifics on running the servers
Server
Configs
Runner
nightly.eclipse-pass.org
docker-compose -f eclipse-pass.base.yml -f eclipse-pass.nightly.yml
nightly script
demo.eclipse-pass.org
docker-compose -f eclipse-pass.base.yml -f eclipse-pass.demo.yml
demo script

For example, to stop all demo services run,
docker-compose -f eclipse-pass.base.yml -f eclipse-pass.demo.yml down

Remove Rogue Processes
If the docker-compose file has drastically changed, then there migth be rogue processes still running. You can view them
with
docker ps
Then grab the container IDs and manually stop them.
docker stop 03159019094f





