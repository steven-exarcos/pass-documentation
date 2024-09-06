# Operations/Production - Eclipse

PASS is an Eclipse Foundation project and benefits the resources and knowledge that the Eclipse Foundation has to offer.
One of those resources is the Eclipse GH repository configuration.

## Eclipse GitHub Repository Configuration

The [.eclipsefdn](https://github.com/eclipse-pass/.eclipsefdn) repository enables the team to
[self-service several aspect of the eclipse organization](https://www.eclipse.org/projects/handbook/#resources-github-self-service)
via a tool called [Otterdog](https://otterdog.readthedocs.io).

These [.eclipsefdn](https://github.com/eclipse-pass/.eclipsefdn) repo / tools gives access to:

* Organization Settings
* Organization Webhooks
* Repositories and their settings
* Branch Protection Rules

Learn more about [Otterdog here](/docs/infra/otterdog.md).

### Adding a New Repository
To add a new repository it requires being a `committer` of the PASS project and performing the following steps: 

* Forking the [eclipse-pass/.eclipsefdn repository](https://github.com/eclipse-pass/.eclipsefdn) into your own GH 
account
* Create a feature branch off the `main` branch following the naming convention of our [developer guidelines](../../community/developer-guidelines.md#pull-request-workflow)
* Modify the `eclipse-pass.jsonnet` with a new repository `orgs.newRepo('REPO_NAME_HERE')`.
  * For the full configuration of the repository see the [Otterdog Jsonnet Configuration section](./ops-eclipse.md#otterdog-jsonnet-configuration)
* Create a PR in the [.eclipsefdn repository](https://github.com/eclipse-pass/.eclipsefdn)
* Include a project lead as a reviewer on the PR.
* One of the release engineers from Eclipse will review the PR and perform the merge.

### Otterdog Jsonnet Configuration
Otterdog stores the GH organization configuration in a [Jsonnet](https://jsonnet.org)

Looking at our current [Jsonnet](https://github.com/tsande16/.eclipsefdn/blob/main/otterdog/eclipse-pass.jsonnet)


### Otterdog Dashboard



