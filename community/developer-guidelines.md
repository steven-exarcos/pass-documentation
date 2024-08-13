# PASS Developer Guidelines

## Getting Involved

* Welcome to the PASS community! There are many ways to participate: trying out the PASS software, letting us know about bugs, suggesting documentation updates, or contributing code. After you’ve read this guide, if you have questions, please send us a message on our [Google Group](https://groups.google.com/g/pass-general), and we will be in touch shortly!
* We primarily use Slack to communicate about PASS development. To be invited to our Slack workspace please send us a message on our [Google Group](https://groups.google.com/g/pass-general).
* Contributing to the project begins as a [contributor](https://www.eclipse.org/projects/handbook/#contributing-contributors) and may lead to being a [committer](https://www.eclipse.org/projects/handbook/#roles-cm). Whether you are a `contributor` or `committer` you will need to sign up for an [Eclipse account](https://accounts.eclipse.org/user/login).
    * A `contributor` can add to and improve PASS by creating issues and submitting pull requests. You’ll find more information about both of these tasks below.
    * A `committer` is an individual who once was a `contributor`, but made significant contributions and was elected by the core team to become a `committer`. They can work directly in the repositories, create and close issues, and merge pull requests.

## Change Request/Bug Report

Would you like to suggest a change to PASS or report a bug? This is done by submitting a GitHub issue.

* When creating an issue to report a bug or suggest a new feature, use the [eclipse-pass/main repository](https://github.com/eclipse-pass/main/issues).
* If available for your particular issue use one of the available [issue templates in the main repository](https://github.com/eclipse-pass/main/issues/new/choose).
* If a template doesn’t exist, use the `Standard Issue` template.
* Add a label if possible, if the label doesn’t exist, or you’re unsure of which one to use, send a message to the  team on the Slack `#pass-dev` channel.
* If possible, suggest a priority. If unsure of the priority, leave it blank and the team will decide what is the priority for the issue.

## Testing

PASS has three different types of tests that are run against the application, and defined by the team as:

* **Unit Tests**: Unit tests focus on a single unit of code. They test very specific conditions, inputs, and expected outputs, validating that the unit behaves as intended. They are narrow in scope, and all other collaborators (e.g. other classes that are called by your class under test) are substituted with mocks or stubs. Unit tests do not guarantee the application will work as intended.
* **Integration Tests**: They test the integration of your application with other parts that are not part of your application e.g. databases, external REST APIs. They are not as narrow as Unit Tests, but still test one integration point at a time. These tests live within the boundary of the application.
* **Acceptance Tests**: They are a final validation step, ensuring PASS fits the workflow requirements for users. The [PASS Acceptance Tests](https://github.com/eclipse-pass/pass-acceptance-testing) runs through workflows using [Test Cafe](https://testcafe.io/) against an instance of PASS. All these tests must pass in order for PASS to be considered production ready.

In general, we recommend the following procedures for testing: 

* If you're planning to submit a code through a Pull Request (PR), please run tests locally first. For Java code, this can be done using `mvn verify` or `mvn clean install`. To test the complete project, [run pass docker](../welcome-guide/setup-run-pass.md) to test.
* If you're planning to submit code which includes new tests, Martin Fowler’s [The Practical Testing Pyramid](https://martinfowler.com/articles/practical-test-pyramid.html) is a great resource for understanding how to structure tests.

### Back-end

* **Unit Tests**
    * If introducing new functionality, please ensure new code is covered by at least one unit test which includes both success and failure states.
        * A few examples of this are [here](https://github.com/eclipse-pass/pass-support/blob/79ad19ed4d2592c342e7cdfdf652a8f7aef3eaa2/pass-deposit-services/deposit-core/src/test/java/org/eclipse/pass/deposit/service/DepositProcessorIT.java#L54) and [here](https://github.com/eclipse-pass/pass-core/blob/e9e853ac7eea05f595fdcd5342ddea99c0798e38/pass-core-main/src/test/java/org/eclipse/pass/object/ElidePassClientTest.java#L84).
    * If performing a bug fix, include a test to ensure that the bug was fixed.
    * In general, unit tests should run quickly.
* **Integration Tests**
    * We recommend running integration tests in a test environment that mimics the production environment as closely as possible.
    * Avoid making network requests to 3rd parties.
         * If needed, use test containers, wiremock, or mockbean.
    * Integration tests should be as fast as possible.
* **Acceptance Tests**
    * Should be based on user requirements and work correctly from a user’s perspective.
    * Updated whenever there are changes to user requirements, significant changes are made to the application, or when they break.
    * Automated so they can be run frequently and consistently.

### UI

* When testing the UI it is helpful to run [Ember locally for faster iteration](https://github.com/eclipse-pass/main/blob/main/docs/dev/running-pass-ui-on-your-host-machine.md).
* Include unit test when you can, such as when functions don't interact with rendering. Otherwise, utilize component integration or ember application/acceptance tests where rendering is involved - this is what ember is best at.
* [Pass-ui](https://github.com/eclipse-pass/pass-ui) is heavy on integration/application tests because it's rendering heavy and much of the business logic is in the back end.
* If you write an encapsulated piece of UI like a component, that component should have at least 1 integration test.
* Application level testing is done with mocked data using Mirage. This needs to be updated diligently, so it doesn't fall out of sync with the real back end. If you are updating the API in a way that changes the contract with pass-ui, please at least create an issue for updating the UI mocking to accommodate these changes.
* At least one test should be added for bug fixes to prevent regression.
* The [pass-acceptance-testing](https://github.com/eclipse-pass/pass-acceptance-testing) acceptance tests are run frequently and can aid in making sure the UI application testing mocked responses are appropriate.
* Ember provides a set of very helpful libraries to assist in testing UI components. It is best practice to use these helpers rather than inventing your own where possible:
    * [Ember test-helpers](https://github.com/emberjs/ember-test-helpers/blob/master/API.md) 
    * [QUnit-dom](https://github.com/mainmatter/qunit-dom/blob/master/API.md) 

## Commits

* As a team, we do not enforce rigid rules about commit messages. However, we strive to write good commit messages by following these [guidelines by Chris Beams](https://cbea.ms/git-commit/).

## Documentation

* We encourage you to read through the [PASS documentation style guide](https://docs.google.com/document/d/11aCooQCNhEq34yG9mGuFynY4xMDRJtPRuOOou9LaCgU/edit?usp=sharing) prior to submitting a pull request.
* The PASS team uses [GitBook](https://www.gitbook.com/) for managing and creating documentation. There are two ways to create new documentation with this system, through the GitBook web interface and through our GitHub `pass-documentation` repository.
* The process for creating, editing, and managing documentation will vary depending on which system you use:
    * GitHub:
        * Use a personal branch that is checked out from `development` and is rebased back into development.
        * Ensure the branch is up-to-date with `development` before creating a pull request.
        * Follow the same pull request guidelines mentioned in our Pull Request Workflow section.
    * GitBook:
        * Request to be added to the GitBook team.
        * Follow the change request process as outlined by the GitBook docs.
             * The same pull request guidelines mentioned in the Pull Request Workflow section apply here as well, such as who should be the reviewer.
             * **NOTE**: It is easy to merge directly from GitBook, ensure that the button at the top right is changed from `merge` to `request a review`.
* Each repository in the PASS project should have a top level README. The following guidelines for these readme should include:
    * Overview of project purpose.
    * Links to appropriate sections in GitBook.
    * README in the main repo should include more details and a longer overview of the PASS project.

## Pull Request Workflow

* If you are in the `contributor` role, you must first fork the repository you wish to make changes in and then submit the pull request to the upstream. If you’re not familiar with pull requests, see the [GitHub pull request documentation](https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/proposing-changes-to-your-work-with-pull-requests/about-pull-requests).
* Pull requests should be reviewed by at least one `committer` that is not the PR author before merging.
    * For pull requests created by `committers`, the pull request author is expected to perform the pull request merge after another committer has approved the pull request.
    * For pull requests created by contributors, the committer reviewing the pull request is responsible for merging the pull request after approval.
* Make sure that the latest version of main is branched for the pull request.
* Branch names should include a ticket number and short description.
    * Example: `978-fix-nihms-loader-etl`
* The description in your pull request should include the following:
    * Summary of major changes and what the pull request will accomplish.
    * Instructions identifying how to test the changes.
* Ensure that every PR is linked to a relevant ticket.
* Update or add new documentation to the `pass-documentation` repository.
    * This would be a separate pull request, see the Documentation section for this process.
* A pull request should not be merged unless all automated checks pass.
* Merges should happen using the rebase strategy.
    * If there have been changes to the `main` code branch, you may want to rebase your branch on `main` for additional safety.
* After a successful merge, delete the branch.

## Pull Request Review Process

* When reviewing the code, here are some advised areas to consider:
  * Verify that the implemented logic aligns with the requirements and specifications.
  * Look for any potential bugs or logical errors.
  * Ensure code is adequately commented where necessary.
  * Verify that sensitive configurations are externalized and not hard-coded.
  * Ensure REST endpoints follow standard conventions (e.g., proper use of [HTTP methods](https://developer.mozilla.org/en-US/docs/Web/HTTP)).
* Review any unit/integration tests and ensure they provide proper coverage.
  * Ensure proper use of mocks and stubs to isolate components during testing.
* Identify and suggest refactoring for any [code smells](https://linearb.io/blog/what-is-a-code-smell).
  * Look for areas that could benefit from improved [design patterns or structures](https://www.baeldung.com/design-patterns-series).
* Ensure that dependencies are properly managed and up-to-date.
  * Check for any potential conflicts or unused dependencies.
* Review commit messages.
* Build the project and run tests locally.

## Closing Issues

* If an issue is linked to a pull request it will auto-close; however for issues that are not linked to a pull request, the  committer performing the merge should close the completed issues.
* Write up the final outcome in the issue and include this as a comment when closing the ticket.
* Link any collaboration or design documents in the issue before closing.
* Ensure all related PRs are linked and closed.
* We recommend that the changes are deployed and tested in a staging or preproduction environment prior to completion.

## Protecting Sensitive Information

* Take precaution to ensure that you’re not committing any credentials/keys/secrets.
* If any sensitive information is accidentally committed, immediately notify the team on the `#pass-dev` Slack channel.
* The core team will triage the severity of the leak and take the appropriate actions, such as removing it from the GitHub and GitBook commit history and rotating the compromised credentials.
* Update GitHub secret scanning to catch any sensitive information that bypassed the original scan.
