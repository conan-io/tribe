# Proposal: New Conan 2.0 lockfiles

| **Status**        |                                                   |
|:------------------|:--------------------------------------------------|
| **RFC #**         | [034](https://github.com/conan-io/tribe/pull/34)  |
| **Submitted**     | 2022-04-26                                        |
| **Tribe votes**   |                                                   |


## Summary

Conan 2.0 will implement completely new lockfiles, with a new proposal of format, definition, behavior, usage and flow, based on the following principles:

- Lockfiles files will be json files containing only three lists of ordered references. One list for “host” requires, another list for “build” requires, another list for “python” requires.
- Lockfiles will no longer contain information about profiles, settings, options (as they did in 1.X)
- Lockfiles will not contain information about the dependency graph, only ordered lists (ordered by version and revision timestamp) of package references.
- The default level of locking will be locking down to the recipe reference, that is, including the version and the recipe revision (``pkg/version@user/channel#recipe_revision``), but not the package-id nor the package-revision. This is aligned with the previously accepted Tribe proposal of removing the [``package_revision_mode``](https://github.com/conan-io/tribe/pull/30). Even if implementing lockfiles locking down the package revision will be possible, that will be considered the exception, and the main flows, documentation and behavior will be optimized for locking down to the recipe revision.
- Locking by default will be non-strict, that is, if some ``requires`` cannot find a matching locked version in the lockfile, it will be resolved normally. A ``--lockfile-strict`` mode will be implemented, but not the default, to enforce finding a match for declared requires in the lockfile or failing otherwise.
- A single lockfile file can lock multiple configurations, for different OS, compilers, build-types, etc., as long as they belong to the same graph. Lockfiles can be constructed for these multiple configurations incrementally, or they can be merged later. The concept of “lockfile bundles” will no longer be necessary
- Lockfiles will not be a version definition mechanism, they need to be a “realizable” snapshot of a dependency graph that should be able to be evaluated in the first place. However, they will allow their usage to define overrides or definitions of versions or recipe revisions that will be used if they fit in the valid definition of the original ``requires`` (that is, if the version fits in the version range of the recipe ``requires``, or always for recipe revisions)
- Lockfiles will be allowed to evolve and adding new information to them easily, at ``conan install``, ``create``, ``graph``, and ``export`` operations, specifying a ``--lockfile-out=newlockfile`` argument. That will allow evolving lockfiles when changes are done to the graph, while keeping control on those changes.

## Motivation

Lockfiles in Conan 1.X were born mainly driven by the need of doing distributed builds in CI, and ensuring the same dependencies were used in this build. They achieved this goal, but not without some pain derived from the fact that the dependency resolution algorithm in Conan 1.X is non-deterministic related to the cache state, that means that 2 different packages with the same dependency and the same version range could resolve to different versions, if some other package in between were forcing the download of a new version in that range from a server.

That resulted in the need to store the full dependency graph in the lockfiles, and as the dependency graph is so tightly coupled to configurations, it required to store one lockfile per configuration, which also added the profiles information, as a "convenience", trying to simplify the management a bit. But locking the full dependency graph made things very challenging, because the algorithm needs to fully identify a match for a package inside that graph, before being able to resolve the locked dependencies. And basically making it almost impossible to do “merge” operations over different lockfiles, and to use a partial lockfile from one graph to resolve other dependencies. It also made necessary to have a very strong locking, because losing the consistency of the dependency graph between locked and unlocked parts was unmanageable.

All of that resulted in lockfiles that have been able to partially address some of the related problems, but that in general are complex to use, understand, and implement CI with them.

So, for Conan 2.0, we are proposing a way more “traditional” approach to lockfiles, with the simplifications enumerated above, making Conan lockfiles something closer to what other package managers like NPM can do. There were a few unknowns if this was doable, so for this proposal we wanted to implement it first, check below for details.

## Detailed Design

In the surface, the basic usage of lockfiles hasn’t change much, creating and using a lockfile would be:

Lets say we have conanfile.txt:

```
[requires]
pkg/[>=0.0 <1.0]
```

And existing ``pkg/0.1`` then:

```bash
conan lock create .  --lockfile-out=conan.lock
# this will create a lockfile with a “requires” list containing “pkg/0.1”
# no matter if other new version pkg/0.2 in the range is created now
# this will keep resolving to “pkg/0.1”
conan install . --lockfile=conan.lock
```

But many things have changed, lets see them.

### Non-strict vs strict

If we now do a modification to our conanfile.txt to:

```
[requires]
pkg/[>=1.0 <2.0]
```

And assuming a ``pkg/1.1`` version exist, then:

```bash
conan install . --lockfile=conan.lock  # will work
```

Will not fail by default, and will be able to resolve to ``pkg1.1``. This is because the default mode is non-strict. One of the most frustrating behaviors of Conan 1.X lockfiles was when lockfiles didn’t allow you to do modifications to your conanfiles and keep working.

If we do:

```bash
conan install . --lockfile=conan.lock --lockfile-strict # will not work
```

It will throw an error saying that the requirement ``pkg/[>=1.0 <2.0]`` cannot be found in the lockfile

### Multi-configuration lockfiles

If we have one conanfile like this:

```python
class Pkg(ConanFile):
    settings = "os"
    def requirements(self):
        if self.settings.os == "Windows":
            self.requires("win/[>0.0]")
        else:
            self.requires("nix/[>0.0]")
```
Then it will be possible to capture the lockfile for both Windows and Linux configuration in a single lockfile file, in the following way, assuming there exist packages ``win/0.1`` and ``nix/0.1``:

```bash
conan lock create .  --lockfile-out=conan.lock -s os=Windows -s:b os=Windows
# That captures a lockfile with ``win/0.1``
conan lock create .  --lockfile=conan.lock --lockfile-out=conan.lock -s os=Linux -s:b os=Linux
# That will augment the conan lock and it will end with both ``win/0.1`` and ``nix/0.1``
```

The later application of the same single resulting “conan.lock” lockfile works for both Windows and Linux configurations:

```bash
conan install . --lockfile=conan.lock -s os=Windows -s:b os=Windows
# resolves to win/0.1, but of course nix/0.1 will not be part of the resulting graph

conan install .  --lockfile=conan.lock -s os=Linux -s:b os=Linux
# resolves to nix/0.1, but of course win/0.1 will not be part of the resulting graph
```

This approach remains valid when different configurations resolve to different versions of the same package. If the package ``conanfile.py`` is:

```python
class Pkg(ConanFile):
    settings = "os"
    def requirements(self):
        if self.settings.os == "Windows":
            self.requires("dep/0.1")
        else:
            self.requires("dep/0.2")
```

When the above steps for both Windows and Linux are executed, the resulting lockfile will contain both ``dep/0.1`` and ``dep/0.2``, including the latest revision for each one, but subsequent ``conan install --lockfile`` installations will still be able to resolve to the correct one, down to the locked recipe revision. This is based on the fact that the locked version still needs to match the required one, so when in Windows, it will look in the lockfile for ``dep/0.1``, will find it there, and will obtain the locked recipe revision from the lockfile.


### Partial lockfiles

Let's say we have a dependency graph ``app``->``pkgc``->``pkgb``->``pkga``.

We can capture a lockfile called “pkgb.lock”, while we are developing ``pkgb``, doing some changes to it, or building it in CI, that locks the dependencies of ``pkgb``, that is it will lock ``pkga/0.1`` Something like:

```bash
# in the pkgb/conanfile, assuming it ``requires = “pkga/[>0.0]”``
conan lock create . --lockfile-out=pkgb.lock
# this pkgb.lock will contain a locked reference to say ``pkga/0.1``
# we can apply this lockfile locking only ``pkga`` all the way down:
# moving to pkgc repo/folder
conan install . --lockfile=pkgb.lock
# this will lock only pkga/0.1, pkgb is not necessarily locked, we didn’t capture it
# moving to app repo/folder
conan install . --lockfile=pkgb.lock
# this will lock only pkga/0.1, pkgb is not necessarily locked, we didn’t capture it
```

In all the above cases, the lockfile contains ``pkga/0.1``, so when executing the different ``conan install --lockfile=pkgb.lock`` in differents ``pkgc`` or ``app`` repos, it will lock exclusively the ``pkga/0.1``, but not the other dependencies in the graph, as they were not added to the graph (it is possible to add them, it will be shown later, but they were not added in this example).

With this in mind, it is then possible to define manually a lockfile with just one single package reference locked, and apply it later to resolve a full graph. Only the defined locked version will be used, while the rest of the graph will be evaluated as always, without locks. 
A ``conan lock add`` command will be provided, so it is more convenient to add things to lockfiles.

Furthermore, it will also be possible to define partial references, that is, even if a lockfile by default captures the ``pkg/version@user/channel#recipe-revision`` full recipe reference, it is still possible to define with ``conan lock add`` a given ``version`` without specifying a recipe revision. In that case, when the lockfile is used, it will use the locked version, and it will resolve dynamically (to latest) the missing piece of recipe revision.


### Modifying lockfiles

Let's say we have a dependency graph ``pkgb``->``pkga``.

We can capture first a lockfile for the dependencies of ``pkgb``, while we are developing it. It will contain only the locked version of ``pkga``. If we decide to ``conan create`` the ``pkgb`` using those locked dependencies, we can augment the existing lockfile to include the recently created ``pkgb`` version (and recipe revision) to the same or to a new lockfile:

```bash
# in the pkgb/conanfile, assuming it ``requires = “pkg/[>0.0]”``
conan lock create . --lockfile-out=pkgb.lock
# this pkgb.lock will contain a locked reference to say ``pkga/0.1``
conan create . --lockfile=pkgb.lock --lockfile-out=pkgb.lock
# now the pkgb.lock contains ALSO the locked pkgb version
```

We have commented above that a ``conan lock add`` operation will be possible, to manually add a version. In the same spirit, it will also be possible to ``conan lock merge`` different lockfiles, as long as those lockfiles belong to the same dependency graph, partially or totally. But they should have shared some common root at some point in time.

These modifications basically add new versions to an existing lockfile, but as such lockfile can be used for different configurations, it doesn’t remove non-used locked references automatically. In order to clean and strip unused references from a lockfile, an explicit ``conan lock create --clean`` creation of a lockfile will be possible.

### Lockfiles in CI

This proposal simplifies the lockfiles and removes some of the commands and functionality that Conan 1.X lockfiles implemented, and that is necessary for implementing CI flows. Conan 2.0 will enable these CI flows with the following functionality:

- ``conan graph build-order`` will output the necessary build-order for a full dependency graph, returning an ordered list of lists, by concurrent-builds levels, in which things need to be built.
- ``conan graph build-order`` will also return the necessary options that every given package in the build-order will need to apply (this information was previously hidden in the lockfile)
- It will be possible to merge different build-orders, even from different products and different dependency graphs with ``conan graph build-order-merge`` which will result in a grouped view, for every package reference, in order, which binaries need to be built.

The idea is that all commands work with the same syntax with and without lockfiles. For example the ``conan graph build-order`` commands above can also be used with and without passing a lockfile at all.

The features designed above will enable 2 flows that were challenging, not to say almost impossible in Conan 1.X, in a straightforward way.

Let’s say we have the ``app``->``pkgc``->``pkgb``->``pkga`` dependency graph, everything is at version ``0.1``, with version ranges like ``[>0.0 <1.0]`` and a developer does a change in ``pkgb`` and bumps its version to ``0.2``. 

Two flows become possible:

1) Starting from ``pkgb`` with a partial lockfile:

- Capture a lockfile for ``pkgb``, locking ``pkga/0.1``, so it can be used to test all different configurations for ``pkgb/0.2`` without risking a concurrent ``pkga/0.2`` is published during this operation.
- Capture a lockfile for ``pkgb/0.2`` while creating ``pkgb/0.2``, including both ``pkgb/0.2`` and ``pkga/0.1``.
- When ``pkgb`` builds correctly, if we want to test the downstream application, we can now capture a new lockfile for it, just feeding the ``pkgb`` one:

  ```bash
  # in the app repository
  conan lock create . --lockfile=pkgb.lock --lockfile-out=app.lock
  ```

- Now use ``app.lock`` to feed it to ``conan graph build-order`` or to ``conan install`` or to ``conan create --build=missing``. 

2) Starting from ``app`` full lockfile:

- Capture the ``app`` product lockfile

  ```bash
  # in the app repository
  conan lock create . --lockfile=app.lock
  ```

- Apply the lockfile for the testing and creation of ``pkgb``, capturing the modified lockfile, to account for the new ``pkgb/0.2``:

  ```bash
  # in the pkgb repository
  conan lock create . --lockfile=app.lock --lockfile-out=app.lock
  ```

- Apply the ``app.lock`` that now includes the new ``pkgb/0.2`` downstream to rebuild ``pkgc`` and ``app`` as needed.

Note that these flows do not require complicated structure or storage of multiple lockfiles. Just 1 single lockfile can be enough for the whole process. If the changes are to be tested for multiple, unrelated products (final consumers), then 1 lockfile for each one might be necessary, but still the complexity will be highly reduced.


## Implementation details

As commented above, the unknowns were many and the uncertainty high, so we implemented this proposal, it is already available in the released alpha.6. It turns, that as the complexity of the lockfiles has been reduced so much, the implementation was also way more straightforward, like one order of magnitude, compared with Conan 1.X lockfiles.

The largest implementation effort had to be done on the preconditions to have this lockfile model, basically:

- To have ordered lists of versions and revisions, it was necessary to attach the revision timestamp to every recipe reference in the whole model. That was a massive change, affecting a very large part of the codebase, and took a while to stabilize.
- It was necessary to implement a different graph resolution algorithm, in which the resolution was deterministic, in a way that it should be impossible to resolve to different versions within a range in the same dependency graph, as that would violate the working hypothesis. As the graph resolution algorithm had to be fully re-implemented to implement the Tribe proposal for requirement-traits and package-types, we took it into account.


## Migration plan

The new lockfiles share nothing with the Conan 1.X lockfiles. The files are different, the commands are different, and the flows that they enable are different. It will be necessary to implement new pipelines in CI for these new flows. The aim of this proposal is that these new CI pipelines for Conan 2.0 will be much simpler than those that already implemented then for 1.X.

There is nothing in the recipes that affect lockfiles, so it will not be necessary to modify any ``conanfile`` to account for this proposal, only command line and lockfiles.
