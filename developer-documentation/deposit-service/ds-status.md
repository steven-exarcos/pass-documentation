# Deposit Services - Statuses

Deposit services primarily acts on three types of resources: `Submission`, `Deposit`, and `RepositoryCopy`. Each of
these resources carries a status. Managing and reacting to the values of resource status is a large part of what Deposit
services does.

Abstractly, Deposit services considers the value of any status to be _intermediate_, or _terminal_.

> It isn't clear, yet, whether this abstract notion of _intermediate_ and _terminal_ need to be shared amongst
> components of PASS. If so, then certain classes and interfaces in the Deposit Services code base should be extracted out
> into a shared component.

The semantics of _terminal_ state are that the resource has been through a workflow of some kind, and has reached the
end of that workflow. Because the workflow has reached a terminus, no additional state is expected to be placed on the
resource, and no existing state of the resource is expected to change.

The semantics of _intermediate_ state are that the resource is currently in a workflow of some kind, and has yet to
reach the end of that workflow. Because the workflow has _not_ reached a terminus, the resource is expected to be
modified at any time, until the _terminal_ state is achieved.

A general pattern within Deposit services is that resources with _terminal_ status are explicitly accounted for (this is
largely enforced by _policies_ which are documented elsewhere), and are considered "read-only".

#### Submission Status

Submission status is enumerated in the `AggregatedDepositStatus` class. Deposit services considers the following values:

* `NOT_STARTED` (_intermediate_): Incoming Submissions from the UI must have this status value
* `IN_PROGRESS` (_intermediate_): Deposit services places the Submission in an `IN_PROGRESS` state right away. When a
  thread observes a `Submission` in this state, it assumes that _another_ thread is processing this resource.
* `FAILED` (_intermediate_): Occurs when a non-recoverable error happens when processing the `Submission`
* `ACCEPTED` (_terminal_): Deposit services places the Submission into this state when all of its `Deposit`s have
  been `ACCEPTED`
* `REJECTED` (_terminal_): Deposit services places the Submission into this state when all of its `Deposit`s have
  been `REJECTED`

#### Deposit Status

Deposit status is enumerated in the `DepositStatus` class. Deposit services considers the following values:

* `SUBMITTED` (_intermediate_): the custodial content of the `Submission` has been successfully transferred to
  the `Deposit`s `Repository`
* `ACCEPTED` (_terminal_): the custodial content of the `Submission` has been accessioned by the `Deposit`
  s `Repository` (i.e. custody of the `Submission` has successfully been transferred to the downstream `Repository`)
* `REJECTED` (_terminal_): the custodial content of the `Submission` has been rejected by the `Deposit`'s `Repository` (
  i.e. the downstream `Repository` has refused to accept custody of the `Submission` content)
* `FAILED` (_intermediate_): the transfer of custodial content to the `Repository` failed, or there was some other error
  updating the status of the `Deposit`

#### RepositoryCopy Status

RepositoryCopy status is enumerated in the `CopyStatus` class. Deposit services considers the following values:

* `COMPLETE` (_terminal_): a copy of the custodial content is available in the `Repository` at this location
* `IN_PROGRESS` (_intermediate_): a copy of the custodial content is _expected to be_ available in the `Repository` at
  this location. The custodial content should not be expected to exist until the `Deposit` status is `ACCEPTED`
* `REJECTED` (_terminal_): the copy should be considered to be invalid. Even if the custodial content is made available
  at the location indicated by the `RepositoryCopy`, it should not be mistaken for a successful transfer of custody.

RepositoryCopy status is subservient to the Deposit status. They will always be congruent. For example, a RepositoryCopy
cannot be `COMPLETE` if the Deposit is `REJECTED`. If a Deposit is `REJECTED`, then the RepositoryCopy must also
be `REJECTED`.

#### Common Permutations

There are some common permutations of these statuses that will be observed:

* `ACCEPTED` `Submission`s will only have `Deposit`s that are `ACCEPTED`. Each `Deposit` will have
  a `COMPLETE` `RepositoryCopy`.
* `REJECTED` `Submission`s will only have `Deposit`s that are `REJECTED`.  `REJECTED` `Deposit`s will not have
  any `RepositoryCopy` at all.
* `IN_PROGRESS` `Submission`s may have zero or more `Deposit`s in any state.
* `FAILED` `Submission`s should have zero `Deposit`s.
* `ACCEPTED` `Deposit`s should have a `COMPLETE` `RepositoryCopy`.
* `REJECTED` `Deposit`s will have a `REJECTED` `RepositoryCopy`
* `SUBMITTED` `Deposit`s will have an `IN_PROGRESS` `RepositoryCopy`
* `FAILED` `Deposit`s will have no `RepositoryCopy`