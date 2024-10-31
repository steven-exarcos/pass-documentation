# PASS Acceptance Tests 
Acceptance / smoke tests for the [PASS application](https://github.com/eclipse-pass). 
This repository utilizes [TestCafe](https://testcafe.io/) to define smoke tests that can run against an instance of PASS.

## Technologies Utilized
* [NodeJS](https://nodejs.org/en/) version 14+
* [Yarn](https://yarnpkg.com/) v1.x - package manager, similar to NPM

## Defining Concepts 

### Acceptance Tests 
Acceptance tests ensure that software aligns with user needs and requirements. The goal of acceptances tests is to evaluate the compliance of the system. Acceptance tests help to determine whether the software is acceptable for delivery.

### Smoke Tests 
Smoke tests are preliminary tests that can reveal simple failures. These tests are designed to catch failures quickly. When smoke tests pass successfully the software is then typically tested with more thorough testing that can take a longer amount of time to complete. Smoke testing can also be referred to as confidence testing, sanity testing, or build acceptance testing.  

### TestCafe
TestCafe is an open-source test runner that is used for end-to-end testing. 

## Running Tests
There are two methods for running acceptance tests for the PASS project. One method is running tests against an instance of PASS that is running locally using pass-docker. The other method is to run acceptance tests against a deployed version of PASS that is running on another system, for exmple running tests against a production, development, or staging environment. The following commands are used to run the acceptance and smoke tests against different environments. 

`yarn run test` - runs the acceptance tests against a locally running pass-docker

`yarn run testDeployment` - runs the acceptance tests against a deployed PASS system such as stage or prod. Please see the `rundeploymenttest.sh` file for required environment variables.
