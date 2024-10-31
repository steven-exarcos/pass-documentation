# Submission
In order to comply with [funder](Funder.md) and institutional access [policies](Policy.md), [Users](User.md) may be required to submit their [Publications](Publication.md) to one or more [Repositories](Repository.md). A Submission is associated with one `submitter` and one Publication. It encapsulates a User satisfying one or more Policies relevant to their Publication by either (1) [Deposits](Deposit.md) initiated in PASS or (2) [Copies](RepositoryCopy.md) of the publication that already exist in the target repositories. The User can start a Submission by describing their Publication and attaching relevant [Grants](Grant.md) to it. The PASS system will use this information to determine which policies apply, and will help the User send the Publication out to the Repositories that will fulfill them.   

Note that the source of a Submission record is not always a PASS User. In some instance, Submissions are created as a result of an import process designed to ensure that the User can see data relevant to their compliance with Policies in a uniform way.

| Field                   | Type    | Description                                                                                                                                                                                                                                                                                                                                                                                                                                                         |
|-------------------------|---------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| id                      | String  | Autogenerated identifier of object                                                                                                                                                                                                                                                                                                                                                                                                                                  |
| metadata                | String  | Stringified JSON representation of metadata captured by the relevant repository forms. This will hold extended metadata relevant to the repositories selected in the Submission workflow. It may include fields such as embargoEndDate, embargoText, pmid, and anything else required by specific repositories etc.                                                                                                                                                 |
| source*                 | String  | Indicates whether the record came from outside of PASS as an import, or was created through the system ([_see list below_](#source-options))                                                                                                                                                                                                                                                                                                                        |
| submitted*              | boolean | When true, this value signals that the Submission will no longer be edited by the User. It indicates to Deposit services that it can generate Deposits for any Repositories that need one. This becomes "true" when the User clicks "submit" in the UI. For Submissions generated by loader processes this value will be false when the User must complete the Submission, or true if it is merely pointing to an existing copy of the Publication in a Repository. |
| submittedDate           | String  | DateTime the record was submitted by the [User](User.md) through PASS                                                                                                                                                                                                                                                                                                                                                                                               |
| submissionStatus*       | String  | The current status of the Submission, derived from the [Deposit](Deposit.md) status(es), [RepositoryCopy](RepositoryCopy.md) status(es) and the `eventType` of the most recent [SubmissionEvent](SubmissionEvent.md) ([_see list below_](#submission-status-options))                                                                                                                                                                                               |
| aggregatedDepositStatus | String  | Current combined status of Deposits, utilized by Deposit Services. The initial status of a new Submission will be "not-started" ([_see list below_](#aggregated-deposit-status-options))                                                                                                                                                                                                                                                                            |
| submitterName           | String  | Name of submitter. This field is used when a preparer nominates a submitter that is not yet a PASS [User](User.md). The name is temporarily stored for use in communications with the submitter until a `User.id` is available. Once there is a URI for `submitter`, the `submitterName` should be null.                                                                                                                                                            |
| submitterEmail          | String  | Email of submitter, formatted as a URI e.g. `mailto:first.last@example.com`. This field is used when a preparer nominates a submitter that is not yet a PASS [User](User.md). The email value is temporarily stored for use in communications with the submitter until a `User.id` is available. Once there is a URI for `submitter`, the `submitterEmail` should be null.                                                                                          |

| Relationship      | Type    | Target                        | Description                                                                                                                                                                                                                                                                                                                                                                                                                                                                                               |
|-------------------|---------|-------------------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| publication*      | To One  | [Publication](Publication.md) | Publication represented in this Submission                                                                                                                                                                                                                                                                                                                                                                                                                                                                |
| repositories*     | To Many | [Repository](Repository.md)   | Repositories that this Publication will exist in when the Submission is completed                                                                                                                                                                                                                                                                                                                                                                                                                         |
| submitter         | To One  | [User](User.md)               | User responsible for submitting the Submission. The User will be the individual who either (a) created this Submission through PASS, thus claiming responsibility; (b) was designated as `submitter` by a `preparer`; or, (c) has been assigned the role based on their being PI of an associated [Grant](Grant.md). When this value is null, it indicates there is not yet a User record for the designated submitter. In this instance there should be a value in `submitterName` and `submitterEmail`. |
| preparers         | To Many | [User](User.md)               | Users who prepared, or who could contribute to the preparation of, the Submission. Preparers can edit the content of the Submission (describe the [Publication](Publication.md), add [Grants](Grant.md), select [Repositories](Repository.md)) but cannot approve Repository agreements, or submit the publication - these tasks must be performed by the `submitter`.                                                                                                                                    |
| grants            | To Many | [Grant](Grant.md)             | Grants that are associated with the User and are relevant to the Publication being submitted                                                                                                                                                                                                                                                                                                                                                                                                              |
| effectivePolicies | To Many | [Policy](Policy.md)           | Policies that will be satisfied via deposit through PASS                                                                                                                                                                                                                                                                                                                                                                                                                                                  |

*required 

## Submission status options

Below are the possible values for the `Submission.submissionStatus` field. They are listed in the order they would typically occur, and with an indication of the arrangement of the data that will result in this status. Note that not all Submissions will go through every status. 

| Value               | Description                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                    | Data determining status                                                                                                                                                                                                                                                                                                                         | 
|---------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| draft               | Newly created Submissions _by the UI_ will have this status by default.  Only unsubmitted Submissions should have this status.  Submissions created by other processes may use a different status.                                                                                                                                                                                                                                                                                                                                                             | Default status for Submissions newly created by a user interacting with the UI.                                                                                                                                                                                                                                                                 |
| manuscript-required | When the PASS system identifies a need for a User to submit a Publication to a particular Repository, it will create a new Submission record with this status in order to prompt the User to provide the document and complete the Submission. For example, PASS imports information from the NIH Public Access Compliance system, which contains information about out of compliance publications - these will appear in the PASS system for PI of the corresponding Grant with the label `manuscript-required`.                                              | New Submissions of this type are created with this status already set                                                                                                                                                                                                                                                                           | 
| approval-requested  | A Submission was prepared by a `preparer` but now needs the `submitter` to approve and submit it or provide feedback.                                                                                                                                                                                                                                                                                                                                                                                                                                          | `Submission.submitted=false` and the most recent [SubmissionEvent](SubmissionEvent.md) has `eventType=approval-requested`.                                                                                                                                                                                                                      | 
| changes-requested   | A Submission was prepared by a `preparer`, but on review by the `submitter`, a change was requested. The Submission has been handed back to the `preparer` for editing.                                                                                                                                                                                                                                                                                                                                                                                        | `Submission.submitted=false` and the most recent [SubmissionEvent](SubmissionEvent.md) has `eventType=changes-requested`.                                                                                                                                                                                                                       |
| cancelled           | A Submission was prepared and then cancelled by the `submitter` or `preparer` without being submitted. No further edits can be made to the Submission.                                                                                                                                                                                                                                                                                                                                                                                                         | `Submission.submitted=false` and the most recent [SubmissionEvent](SubmissionEvent.md) has `eventType=cancelled`.                                                                                                                                                                                                                               | 
| submitted           | The submit button has been pressed through the UI. From this status forward, the Submission becomes read-only to both the `submitter` and `preparers`. This status indicates that either (a) the Submission is still being processed, or (b) PASS has finished the Deposit process, but there is not yet confirmation from the Repository that indicates the Submission was valid. Some Submissions may remain in a `submitted` state indefinitely depending on PASS's capacity to verify completion of the process in the target [Repository](Repository.md). | `Submission.submitted=true`, and the Publication associated with the Submission is in a positive status for each Repository i.e. it's `RepositoryCopy.copyStatus` is not `rejected` or `stalled`, and in the absence of a RepositoryCopy, the `Deposit.depositStatus` is not `rejected`.                                                        |
| needs-attention     | Indicates that a [User](User.md) action may be required outside of PASS. The Submission is stalled or has been rejected by one or more [Repository](Repository.md)                                                                                                                                                                                                                                                                                                                                                                                             | The `copyStatus` of one or more [RepositoryCopy](RepositoryCopy.md) for the Submission is `rejected` or `stalled`. In the absence of a `RepositoryCopy`, the [Deposit](Deposit.md) for that Repository has a `depositStatus` of `rejected`. To be clear, a positive status on the RepositoryCopy can override a negative status on the Deposit. | 
| complete            | The target repositories have all received a copy of the Submission, and have indicated that the Submission was successful.                                                                                                                                                                                                                                                                                                                                                                                                                                     | There is a RepositoryCopy with `repoCopyStatus=complete` for each of the target repositories.                                                                                                                                                                                                                                                   |

## Aggregated Deposit status options

These are the possible statuses for a Submission's aggregatedDepositStatus field. They are listed in the order they would occur. Note that not all Submissions will go through every status. 

| Value       | Description                                                                                                                                      |
|-------------|--------------------------------------------------------------------------------------------------------------------------------------------------|
| not-started | No [Deposits](Deposit.md) have been initiated for the Submission                                                                                 |
| in-progress | One or more [Deposits](Deposit.md) for the Submission have been initiated, and at least one has not reached a status of "accepted" or "rejected" |
| failed      | One or more [Deposits](Deposit.md) for the Submission has a status of "failed"                                                                   |
| accepted    | All related [Deposits](Deposit.md) for the Submission have a status of "accepted"                                                                |
| rejected    | One or more [Deposits](Deposit.md) for the Submission has a status of "rejected"                                                                 |

## Source options

These are the possible sources of a Submission

| Value | Description                                                                                                 |
|-------|-------------------------------------------------------------------------------------------------------------|
| pass  | Submission record was created or submitted via the PASS user interface                                      |
| other | Submission record was automatically created by harvesting and ingesting from a 3rd party service e.g. NIHMS |