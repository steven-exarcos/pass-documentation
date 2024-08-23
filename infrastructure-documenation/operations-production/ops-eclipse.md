### Operations/Production - Eclipse

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
