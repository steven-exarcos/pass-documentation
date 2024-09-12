# Operations/Production - Eclipse Demo Site

The Eclipse Demo website is used to demonstrate PASS to the community and potential adopters of PASS. The demo website
is hosted at [demo.eclipse-pass.org](https://demo.eclipse-pass.org). This site is hosted from our JHU AWS infrastructure.
It has a different infrastructure from our production site as it's only running a container of Pass Core and Pass UI.
Additionally, the demo site is loaded with fake data for demonstration purposes.

The demo site uses a login page as opposed to using SSO integration. To login use the following fake username and 
password: `nih-user` and `moo`. There are other types of users that can be tested in the demo environment and their 
logins are the same as the local docker setup. A list of these test accounts can be found in the Welcome Guide 
[Setup and Run PASS Locally](../../welcome-guide/setup-run-pass.md#run-pass-locally).