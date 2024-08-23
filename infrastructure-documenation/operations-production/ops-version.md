# Operations/Production - PASS Versioning

### PASS Release Versioning

## TODO edit this so it reflects the current Release versioning scheme

A release of PASS is made up of Java artifacts, Node modules, and Docker images. Maven builds all the Java artifacts and
many of the Docker images. The Node based pass-ember adapter is released to the NPM registry. The Node based pass-ui is
released as a Docker image in pass-docker. The Maven releases are the most complicated part of the overall process.

In this approach all the maven modules are composed together into a single hierarchy such that they can all be
processed by the Maven reactor and the Maven release plugin used. This allows all the Java modules to be released with a
single command. Note that for every project with a different version, the development and release versions must be
passed as arguments or provided interactively. Also the Maven reactor won’t be able to figure out the dependencies
introduced by integration test Docker images unless they are specified in the project poms.

There are a few approaches that might be used to assemble this super module. The main pass repository which hosts the
root PASS pom could have all our Maven repositories listed as Maven submodules by adding each corresponding repository
as a Git submodule. Some investigation makes it look like this should work. The release plugin needs to get configured
to handle the submodules when it does a checkout. But more generally this seems a little clunky and complicated. A lot
gets linked into our main repository.

Another possibility is writing a script which assembles all the Maven repositories to be released into a hierarchy with
the root PASS pom at the top and edit the root pom to have them as submodules. But this won’t work because the code to
be released must be checked into a repository. We can workaround that problem by creating our own local Github repo and
doing the release against that. The local repository would have exports of every other PASS maven repository main
branch. That leaves a final problem of getting the changes made to the local repository (version changes and tags) into
the corresponding upstream repositories. This seems awkward, but doable.
Release separately

Instead of assembling a super pom and using the Maven reactor to handle dependencies between PASS projects, a script
could just release each project separately. This must be done in dependency order which would be hard coded into the
script. The dependency order is not hard to figure out. See above for an attempt. If every PASS project is on a
different version, the script would have to manage all of those versions. In addition given, that we would be going
through a list of modules in a certain order, adding support for resumption would not be difficult. Overall this seems
like the simplest approach.

Decisions and recommendations
The team has decided to use a single version across all of PASS. This has a number of benefits, not the least of which
is ease of understanding. Java projects and Docker images should be moved to the new version which is 0.1.0-SNAPSHOT.

I recommend that the Maven release process does not use a super module and instead releases projects separately in
dependency order.