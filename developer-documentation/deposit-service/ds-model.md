# Deposit Services - Model

The Deposit Services data model describes the interaction between Deposit Services and the PASS data model, detailing 
how various resources, such as `Submission`, `Repository`, `Deposit`, and `RepositoryCopy` are managed. Additionally, it 
outlines the internal data model, configuration, packaging, and transport mechanisms used by Deposit Services to 
facilitate the transfer and validation of submissions to downstream repositories.

## PASS Model

Deposit Services uses objects in the PASS data model, which are distinct from the Deposit Services' internal model.
Objects in the PASS data model are persisted in the PASS core service. Thus, any interaction
with PASS resources will require CRUD operations (using the PassClient) by Deposit
Services.

PASS objects used by Deposit Services are:

* `Submission`: Read by Deposit Services, updates `Submission.AggregatedDepositStatus`.
* `Repository`: Only ever read by Deposit Services, never modified.
* `Deposit`: Created and modified by Deposit Services.
* `RepositoryCopy`: Created and modified by Deposit Services. Note that the NIHMS loader also creates
  `RepositoryCopy` resources.

<figure>
  <img src="../../.gitbook/assets/pass-model.png" alt="Deposit Services PASS Model">
  <figcaption>
    <p>Deposit Services PASS Model</p>
  </figcaption>
</figure>

Each `Submission` resource links to one or more `Repository` resources. Deposit Services will create a `Deposit`
and `RepositoryCopy` resource for each `Repository` linked to the `Submission`. Deposit Services will attempt to deposit
to the downstream system represented by the `Repository`. The status of a deposit to a downstream repository will be 
recorded on the `Deposit` resource. That is to say, the `Deposit` records the transaction and its success or failure 
with a `Repository`, and the `RepositoryCopy` records where the Repository stored the content of the `Deposit`.

## Deposit Services Internal Model

Deposit Services has an internal object model that is distinct from the PASS data model. Instances of the internal 
object model are not persisted in the PASS repository, or anywhere else. Upon receipt of a `Submission` (i.e. the 
external PASS model), Deposit Services immediately converts it to an instance of the internal model using a Model
Builder.

<figure>
  <img src="../../.gitbook/assets/ds-model.png" alt="Deposit Services Internal Model">
  <figcaption>
    <p>Deposit Services Internal Model</p>
  </figcaption>
</figure>

### Deposit Model

* `DepositSubmission`: internal representation of a PASS Submission.
* `DepositMetadata`: metadata describing the submission, parsed from the "metadata blob" (**`Submission.metadata`**)
  and other `Submission` properties.
* `DepositSubmission` is the central entity in the internal DS model. It brings entities and properties of the public PASS
  model into a model specific to producing a package. Many of the fields or classes are bibliographic in nature, with the
  `DepositFile` linking to the binary content of the submission (the files uploaded by the end user in the submission
  process). A secondary purpose of the internal DS model is to surface bibliographic metadata explicitly, since a 
  primary responsibility of DS is to map bibliographic metadata to package metadata. Hard-coding bibliographic metadata
  in the internal model could be considered an anti-pattern.

### Configuration Model

* `Packager`: encapsulates configuration of the `Assembler`, `Transport`, and `DepositStatusProcessor` for every
  downstream repository in `repositories.json`. Each repository configured in `repositories.json` should to reference
  a `Repository` resource in the PASS repository.
* `RepositoryConfig`: Java representation of a single repository configuration in `repositories.json`. The
  configuration for a repository includes directives for the transport protocol used for deposit (including
  authentication credentials), packaging specification used for deposit, and packaging options.

<figure>
  <img src="../../.gitbook/assets/config-model.png" alt="Deposit Service Configuration Model">
  <figcaption>
    <p>Deposit Service Configuration Model</p>
  </figcaption>
</figure>

* Each downstream repository is represented in the public PASS model as a `Repository`; each `Repository` carries a unique
  key, which is a short, human-readable string (e.g. `jscholarship`, `dash`, `pmc`). Each `Repository.key` is represented in
  the DS configuration model as `RepositoryConfig.repositoryKey`. When a Submission is processed, the configuration for the
  Repository is resolved by its key.

### Packaging Model

* `Assembler`: responsible for creating and streaming the content (i.e. the files uploaded by the end-user and any
  metadata required by the packaging specification) of a Submission according to a packaging specification.
* `PackageStream`: the content of a `Submission` to be deposited to a downstream repository as a stream, as opposed 
  to bytes held in a buffer or stored on a file system.
* `Transport`: an abstraction representing the physical protocol used to transfer the package stream from the PASS
  repository to the downstream repository.

### Messaging Model

* `DepositStatusProcessor`: responsible for updating the `Deposit.depositStatus` property of a `Deposit` resource,
  typically by resolving the URL in the `Deposit.depositStatusRef` property and parsing its content.
* `CriticalRepositoryInteraction`: CRI for short. Performs an optimistic locking (`If-Match` using an Etag) "critical"
  modification on a PASS resource, with a built-in retry mechanism when a modification fails. Each CRI has a
  pre-condition, critical section, and post-condition. The pre-condition must be met before the critical section is
  executed. The post-condition determines whether the application of the critical section was successful. The built-in
  retry mechanism re-uses the pre/post/critical functions in the case of a conflict.

## Model Builder

Upon receipt of a Submission, the `DepositSubmissionModelBuilder` is invoked to produce an instance
of `DepositSubmission`. The `DepositSubmissionModelBuilder` accepts a `Submission` ID for conversion to 
`DepositSubmission`.

## Assembler

Responsible for assembling the content of a submission into a streamable package (i.e. the `Assembler` returns
a `PackageStream` instance). This includes:

* Resolving the custodial content being deposited from the PASS repository.
* Generating any metadata required by the packaging specification.
* Generating any metadata required by the downstream repository.
* Encapsulating all of the above into a stream of bytes that meets a packaging specification.

Implementing the Configurable Metadata Framework focuses on the support of pluggable Assemblers within Deposit
Services; different `Assembler` implementations can include metadata required for their repository.

### Custodial and supplemental resources

The term _custodial resource_ is used throughout: a custodial resource is content that was uploaded by the end user for
deposit to a downstream repository, such as their data sets, manuscripts, etc. Non-custodial resources (i.e. _supplemental
resources_) include metadata describing the content, for example, BagIt tag files or DSpace METS XML files.

#### Abstract Assembler and Archiving Package Stream

There are two abstract classes to help developers create `Assembler` implementations. The `AbstractAssembler`
and `ArchivingPackageStream`. There is also a concrete class named `SimplePackageStream` that can be used for 
non-archive integrations.

The `AbstractAssembler` contains shared logic for building a list of custodial resources to be deposited. Concrete
implementations accept the list of custodial resources (among other parameters, including the packaging specification)
and produce the `PackageStream`.

The `ArchivingPackageStream` contains shared logic for assembling multiple files into a single zip, tar, or tar.gz file.

The `SimplePackageStream` contains the associated `DepositSubmission`, List of `DepositFileResources`, and returns
metadata to be sent to repository.

#### MetadataBuilder and PackageStream.Metadata

`PackageStream.Metadata` is an interface that provides package-level metadata. The `MetadataBuilder` is a fluent API for
creating physical package-level metadata, such as:

* Packaging specification - a URI identifying the package specification used.
* Package size (bytes) and its checksum.
* The package name.
* Mime type, compression used, and archive format.

The `PackageStream.metadata()` method returns the `PackageStream.Metadata` for a `PackageStream` instance. Because some
metadata is unknown prior to streaming (e.g. the package size), the metadata returned by this method may be incomplete
until after the stream has been read.

#### ResourceBuilder and PackageStream.Resource

`PackageStream.Resource` is an interface that provides metadata describing a resource within the
package. `ResourceBuilder` is a fluent API for creating physical metadata describing each resource (i.e. file) within
the package:

* File size (bytes)
* Filename, including its path relative to the package root
* Checksum and mime type

The `PackageStream.resources()` method answers an `Iterator` over each `Resource` in the `PackageStream`.

#### Package Provider

`PackageProvider` is an interface that is invoked by Deposit Services when a `PackageStream` is streamed to a repository
via a `Transport`.

`PackageProvider` represents a streaming lifecycle interface that has three methods: `start(...)`, `packagePath(...)`,
and `finish(...)`. The `start(...)` method is invoked after the custodial resources have been assembled, but before
streaming has started. The `packagePath(...)` method is invoked prior to streaming each custodial resource.
The `finish(...)` method is invoked after all the custodial resources have been streamed, and provides an opportunity
for the `PackageProvider` to add supplemental resources to the package being streamed.

For example, a BagIt `PackageProvider` would ensure that each custodial resource is pathed under `<package root>/data`
when implementing `packagePath(...)`. After the custodial resources are streamed, the BagIt `PackageProvider` would
assemble and stream all the BagIt metadata: bagit.txt and any other tag files.

## Transport

Responsible for transferring the bytes of a package (i.e., a `PackageStream`) to an endpoint. The Transport API is
designed to support any transport protocol. Each downstream repository in `repositories.json` must be configured with
a `Transport` implementation.

The `Assembler` and the `PackageProvider` create the package, and the `Transport` is the "how" of how a package is
transferred to a downstream repository. Choosing the `Transport` to be used depends on the support of the downstream
repository for things like S/FTP, SWORD, InvenioRDM (HTTP) or other protocols.

For example, a BagIt `Assembler` and `PackageProvider` would produce BagIt packages. Those packages may be transported
to downstream repositories using S/FTP, SWORDv2, or a custom Transport implementation. The `Transport` to be used is a
matter of configuration in `repositories.json`.

### S/FTP

Supports the transport of the package stream using S/FTP.

#### SWORDv2

Supports the transport of the package stream using the [SWORD protocol version 2](http://swordapp.github.io/SWORDv2-Profile/SWORDProfile.html).

#### InvenioRDM

Supports the transport of the package stream that will be sent to a InvenioRDM repository. This implementation uses
HTTP to call the InvenioRDM REST API.