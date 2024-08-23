# Operations/Production - PASS Versioning

A release of PASS is made up of Java artifacts, Node modules, and Docker images. Maven builds all the Java artifacts and
many of the Docker images. The Node based pass-ember adapter is released to the NPM registry. The Node based pass-ui is
released as a Docker image in pass-docker. The Maven releases are the most complicated part of the overall process.

There is a single version of PASS across all components. We've decided to take this approach due to several benefits,
but the most important reason is the ease of understanding. PASS uses semantic versioning following this convention:

```
X.Y.Z-[other-labels] 
```

Where,

* X = Major updates
* Y = Minor updates
* Z = Patches (bug fixes)

The other labels include:

* SNAPSHOT, the current development release.
* RC[X] e.g. RC1 or RC2, which stands for Release Candidate 1, Release Candidate 2 etc.

The SNAPSHOT label is generated for the next development version and happens automatically in our CI/CD pipeline. When a
release is deployed, a SNAPSHOT is automatically created during the release. The release candidate label is preparing a 
release to be deployed before the final version is completed. Bug fixes are the only updates permitted in a release 
candidate version. A couple of examples below demonstrate the usage of this versioning scheme:

* Release version:
```
1.9.0
```
* Patch:
```
1.9.1
```
* Development version:
```
1.10.0-SNAPSHOT
```
* Release Candidate 1:
```
1.10.0-RC1
```
