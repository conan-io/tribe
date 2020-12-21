
# Proposal: Recipe and Package immutability

| **Status**        |                            |
|:------------------|:---------------------------|
| **RFC #**         |                            |
| **Submitted**     | 2020-12-18                 |
| **Tribe votes**   |                            |

---


## Summary

Packages and recipes, identified by their complete reference including the revision, all artifact bare file contents (conanfile.py, source files, compiled libraries and executables, etc) stored in the Conan cache and in servers are always immutable.


## Motivation

It is a best practice and well known principle in package management and devops that packages, once created, should never change.

This was not the Conan behavior when not using revisions, and the same recipe reference (pkg/version@user/channel), or the same package reference (pkg/version@user/channel:package_id) could store different artifacts, new packages with new changes could be created that will overwrite existing ones in the cache and in servers.

Introducing revisions greatly improve this, introducing a new “coordinate” into the recipe and package reference. The revision being a hash of the contents of the recipe or package guarantees the uniqueness and immutability of such recipe or package.

However, there is still a corner case in Conan 1.X that violates this principle, and this proposal aims to remove this behavior in Conan 2.0. Also, this immutability has not been exploited yet in Conan to achieve better performance and more straightforward processes in uploads, downloads and updates from servers.

## Proposal

Recipes and packages will always be immutable, and will not be able to change. The immutability also refers to the references, the “name” or “coordinates” that define one recipe or one package binary in the system which are:

- Recipe reference: ``pkg/version@user/channel#recipe_revision``
- Package reference: ``pkg/version@user/channel#recipe_revision:package_id#package_revision``

When one recipe or package is created, it will be assigned such a reference, which will not be possible to change after that.

The immutability applies to the actual file contents: the bytes stored in the different raw files: conanfile.py, C++ sources and headers, compiled libraries (.lib, .a, .so), executables (.exe), etc. This does not apply to storage artifacts (the conan_sources.tgz or the conan_package.tgz) that might be used to communicate and store with the servers. It also does not apply to file metadata, as owner/group or file mode, as this can easily change between different OSs and machines, and those changes would render the revisions unusable.


## Detailed Design

This immutability can be achieved if:

- Revisions are always used. This is already approved and merged by the Tribe, and Conan 2.0 will be only using revisions.
- Revision based on the hash of the contents are good, they are already immutable by definition (doing a change will produce a new revision)
- Revisions based on the SCM commit (revision_mode = "scm") that are “dirty=not everything ignored or committed) will completely block the upload of the recipe or package. No possibilities of workarounds. Even if possible to create a package locally (for debugging or working purposes) in a dirty state, no provision or functionality based on the immutability principle will be provided, and at all effects the package will be considered as unchanged. Users using revision_mode = "scm" are strongly encouraged to check their flows, and do not rely on creating packages with a SCM dirty state
- The ``conan copy`` command will be removed, as it can mutate a package reference.

Immutability will be exploited by Conan processes. These are just some examples, this doesn’t aim to be a complete list of all the commands, processes or behavior that will be able to be improved based on the immutability principle. These will be discussed later, and specifically for each one.

- If the upload detects a revision already exists in a server, it can completely skip the upload, no need to check the contents of the server, it will be assumed identical.
- Updating and getting the latest revision or version of packages will not require to check the package contents (conanmanifests.txt) or the timestamps, and just the revisions and timestamps of the revisions will be considered for resolving.
- The concept of “recipe outdated” disappears. It is not possible to have some binaries that are outdated wrt the recipe. The full package reference is “pkg/version@user/channel#recipe_revision:package_id#package_revision”. If the recipe changes, it will require new binaries.
