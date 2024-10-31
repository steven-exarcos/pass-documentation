# Operations/Production - Eclipse Operations

PASS is an Eclipse Foundation project and benefits from the resources and knowledge that the Eclipse Foundation has to 
offer. One of those resources is the Eclipse GH repository configuration.

## Eclipse GitHub Repository Configuration

The [.eclipsefdn](https://github.com/eclipse-pass/.eclipsefdn) repository enables the team committers to
[self-service several aspect of the eclipse organization](https://www.eclipse.org/projects/handbook/#resources-github-self-service)
via a tool called [Otterdog](https://otterdog.readthedocs.io).

These [.eclipsefdn](https://github.com/eclipse-pass/.eclipsefdn) repo / tools gives access to:

* Organization settings
* Organization webhooks
* Repositories and their settings
* Branch protection rules

Learn more about [Otterdog here](/docs/infra/otterdog.md).

## Eclipse Contributor Agreement and Eclipse Development Process

Contributors of the project must electronically sign appropriate documents in order to become committers. The following
are agreements and policies by Eclipse that a committer must read:

* [Eclipse Contributor Agreement (ECA)](https://www.eclipse.org/legal/eca/)
* [Eclipse Development Process (EDP)](https://www.eclipse.org/projects/dev_process/)

## ECA for Pass Documentation

ECA is not configured as a required check for merging in `pass-documentation`, therefore PRs can be merged with a 
non-committer. In addition, the EDP explicitly states: "you can merge if you know that the user associated with the 
commit has signed an ECA", therefore if a user with a different GitHub account with a different email address from their
Eclipse committer account, is still able to merge PRs with those commits. This applies to GitBot (GitBook GitHub bot), 
and Eclipse is aware of this account to make commits.