# Deposit Services - Next Steps / Institution Configuration

It is important to understand that the Deposit Service is not overly prescriptive. When PASS was developed, we did not
have a set of requirements for supporting a wide range of repositories, but we acknowledged that possibility existed. So
a balance was struck with the Deposit Service APIs: if they were over-specified, there may be use cases or repositories
that couldn't be accommodated later. If they were under-specified, then Deposit Service wouldn't be able to provide
value by sharing implementations or behaviors between repositories.

Since an exhaustive analysis of repository interfaces was impractical at the time, the idea was to provide general
interfaces that could accommodate almost any scenario. As more use cases, patterns or abstractions were revealed (by
onboarding new institutions) they could be formalized (e.g. as Java interfaces).

Therefore, supporting new repositories or use cases requires novel code to be developed.

## Supporting a New Repository

### High Level Requirements

In order to support a new downstream repository (e.g. Dataverse, Islandora), there are four high-level requirements to
negotiate:

1. Repository protocol: the protocol used to transmit the bytes to the repository (e.g., S/FTP, SWORD).
2. Custody transfer: mechanism used to determine whether a successful custody transfer took place.
3. Package spec: governs physical characteristics of the package (structure, pathing, naming), and required or optional
   metadata (e.g., NIH bulk package spec, DSpace METS, BagIT).
4. Metadata mapping: maps elements of the PASS model to package metadata elements.

The philosophy of the Deposit Service is that these requirements are a negotiation. This is for a few of reasons:

* PASS does not want to prescribe or dictate to a downstream repository what it must accept.
* Network effects are not in PASS' favor. A popular repository like DSpace is not going to change its package 
* Requirements to accommodate limitations in PASS.

In order to satisfy their workflows, institutions will place requirements on packages, especially the metadata contained
in the package. Universities, the NIH, or the NSF are not going to change their package requirements to conform with PASS.

### Rationale for Packages

The one thing PASS cannot negotiate is the package-oriented nature of the Deposit Service. The concept of a package is
baked into the Assembler and Transport API. As a consequence, the following kinds of interactions are not supported by
Deposit Services:
* Transfer of discrete files: the Deposit Service does not transfer the files of a Submission one by one. Files submitted
  to PASS by an end user are rolled up and transferred in a single archive file (i.e., a package).
* Mapping to downstream data models: the Deposit Service does not, for example, create DSpace Communities, Collections, or
  Items. It does not create LDP resources in Fedora. It does not create Islandora Nodes or Media. It will send a package,
  and the downstream repository is responsible for receiving the package and mapping from the package model to the native
  repository model.

Packages are a shared concept that significantly reduce the number of repositories supported. Theoretically, if PASS 
provides generalized support for popular package models, interoperability with a variety of downstream repositories will 
be more easily achieved.

### Interface Overview

If implementing a new repository the following interfaces will need to be implemented:

* Implementation of Assembler API
* Implementation of PackageProvider API
* Implementation of Transport API
* If repository exposes custody status, implement DepositStatusProcessor

### Downstream Requirements

The downstream repository must be able to receive a package, and ingest its content:

* Unpack the package,
* interpret its content,
* map it to the native repository model,
* and store the bytes of each file.

Ideally a downstream repository will have some mechanism to determine and expose the status of custody transfer (i.e. a
package has been accepted or rejected), but that is not a strict requirement. PASS will still function even if the
downstream repo doesn't expose the status of a deposit attempt.

### Repository Protocol

The repository protocol is primarily implemented by the Deposit Service Transport API. The Transport API is responsible
for using a given protocol to connect to the downstream repository, initiate a transfer of bytes (the package), and
interpret the response for success or failure. In some cases, the response carries valuable information that must be
persisted (e.g., Deposit.depositStatusRef is captured from a SWORD response for the DepositStatusProcessor to act on
later).

The transport implementation is also responsible for indicating the location of the package, e.g. the folder (FTP) or
collection (SWORD) the package will be transferred to within the downstream repository. These transport hints are
supplied as key-value pairs to the underlying implementation.

If the repository protocol is SWORDv2 or FTP, the existing implementation may be reused, otherwise write your own.

If the repository has a use case that is not handled by an existing implementation, it might be accommodated by
supporting new transport hints for an existing implementation rather than writing a novel implementation. Supporting an
additional transport hint involves:

* Adding and documenting the hint in the appropriate class.
* New logic to implement hint behavior.
* Writing tests for the new behavior.

The `Sword2TransportHints.SWORD_COLLECTION_HINTS` and the corresponding logic in
`Sword2TransportSession.selectCollection(...)` are a good example of accommodating a new use case within an existing
implementation for which there is no formal abstraction.

### Custody Transfer

Determining the status of custody transfer is the responsibility of the `DepositStatusProcessor` interface.

- Determining the success or failure of transfer of custody is an async process.
- The downstream repository may have a workflow for validating the content of the package before accepting it.
- The Deposit Service makes no assumption about how long that process may take, or in what form, or if it will ever
  complete.
- A process must run periodically to process PASS Deposit resources that have an intermediate deposit status (i.e.
  deposits for which custody transfer has not been confirmed)
- The process is represented by the `DepositStatusProcessor` interface.

If the repository protocol is SWORDv2, then the `DefaultDepositStatusProcessor` may be used. Otherwise a novel
implementation is required.

As an example, the `DefaultDepositStatusProcessor` consults the `Deposit.depositStatusRef` and assumes that reference is
URI to a SWORD Statement, which provides the real-time status of the package in the repository.

If the downstream repository rejects custody of the package, the only recourse is for the end user to perform another
submission that addresses the reason(s) for the rejection. The reasons for rejecting a package are not communicated.
PASS does not facilitate this.

In practice, a faculty member would need to:
1. understand that their submission has been rejected
2. know who to reach out to, e.g. a support interface or phone number
3. work with PASS support staff to initiate a new submission that addresses the reasons for rejection

PASS support staff would need to comb through logs, contact the downstream repository administrator, or take other
action to determine the cause of the rejection.  PASS has no support for remediating submissions that failed because
the downstream repository worklow rejected it.

### Package Specification

Implementation of the Deposit Service `Assembler` and `Package Provider API`. If the packaging spec is NIHMS or BagIT, reuse
existing implementations. If the packaging spec is DSpace METS, reuse existing implementation, and provide a metadata
mapping in a package provider. If a new specification is being supported, write a new implementation of the Assembler,
using shared implementations like AbstractAssembler and ArchivingPackageStream.

Concrete subclasses of AbstractAssembler are responsible for implementing the PackageProvider interface.

The complexity of creating a stream is encapsulated in ArchivingPackageStream, which is shared across all Assemblers
that produce a single archive file. They have allowed for the generation of packages for BagIt, DSpace METS, NIHMS,
along with simple package formats used for integration tests.

#### Important Concepts in the Assembler API:

- Custodial content: content supplied by the end user in a PASS Submission; these are PASS File entities.
- Supplemental files: not to be confused with the PASS File role option of the same name; these are files that are
  generated by the Assembler, usually to comply with a package specification. E.g. a manifest of files and their
  checksums, or a file containing metadata for the package. End users do not upload supplemental files, they are supplied
  by the Assembler implementation.
- Package Provider (discussed in the next topic): responsible for producing the supplemental files included in a package
- Packages are streams, and are designed to be relatively performant. For example, a client of the Assembler API can open
  a PackageStream and begin reading from it right away, and the stream won't block as long as the implementation continues
  to supply bytes.

#### Important Abstractions in the Assembler API

- `Resource` and `ResourceBuilder`: A Resource carries metadata about an individual file in a package: its size, media type,
  file name, and checksums. The ResourceBuilder allows a Resource to be built as the PackageStream is written, and
  different components may contribute to a Resource during this process. For example, one component that determines the
  media type of `Resource` that will need to be invoked at the beginning of streaming the resource (typically the first 
  512 bytes are used). Another component may be invoked after streaming the resource to calculate its checksum. The
  ResourceBuilder allows different components to contribute to the Resource state without sharing the concrete
  implementation.
- `MetadataBuilder` and `Metadata`: Metadata and MetadataBuilder are similar to Resource and ResourceBuilder, except 
that they provide for the package metadata, as opposed to individual files within the package.
- `StreamWriter` and `DefaultStreamWriter`: TODO need content

#### Metadata Mapping

Implementation of the Deposit Service Package Provider API. Specifically the generation of Supplemental Resources;
metadata files that are generated by the Deposit Service, not submitted by the end user. Examples of supplemental
resources include:
* BagIt
  * bagit.txt: a required file in BagIt packages identifying its version and encoding
  * manifests: a required file in BagIt packages listing the contents and their checksum
  * bag-info.txt: an optional file in BagIt (though often required by institutional profiles) that provides descriptive
      metadata about the package; this is the volatile section of BagIt
* DSpace METS
  * METS.xml: a required file in DSpace METS packages that contains a variety of metadata; the `<dmdSec>` is the volatile
      section of that document.