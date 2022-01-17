# Proposal: Deprecate "package_revision_mode"


| **Status**        |                                                   |
|:------------------|:--------------------------------------------------|
| **RFC #**         | [030](https://github.com/conan-io/tribe/pull/30)  |
| **Submitted**     | 2022-01-17                                        |
| **Tribe votes**   |                                                   |


## Summary
Remove the ``package_revision_mode`` from the ``default_package_id_mode`` and from the ``package_id()`` method definitions. The package revision will never be part of the "package_id" of consumers.


## Motivation
When we realized that the default computation of the "package_id" in Conan 1.X was not enough for the majority of cases, different strategies to compute the "package_id" were introduced.

Initially the implemented strategy was "semver" mode, which, according to the semver specification, meant that only changes to the major version were breaking. Accordingly, such default "semver_mode" converted the dependencies from ``zlib/1.2.11@user/table`` to ``zlib/1.Y.Z``, and that was the actual string being hashed. That means that using a minor or patch version of zlib, like ``zlib/1.2.12`` if it existed, will result in the same "package_id" of the consumer, and thus the same binary.

New modes like "minor_mode", mapping to ``zlib/1.2.Z`` or "patch_mode", mapping to ``zlib/1.2.11`` were defined. Likewise  "recipe_revision_mode" uses the full ``pkg/version@user/channel#revision`` into the "package_id" computation. In practice, this means that every change to the source code (both conanfile.py or source code) of zlib, produces a new recipe revision of ``zlib``, and that will define a new ``package_id`` of every other package depending directly or indirectly on zlib, requiring a new binary of those consumer packages to be built from sources. This is a very strong guarantee: every change in any recipe always requires a build and a new binary from all the consumers..

But then, the ``package_revision_mode`` was also defined, adding the full package reference, down to the package revision, into the ``package_id`` of the consumers, hashing the full ``pkg/version@user/channel#recipe_revision:package_id#package_revision``. This introduced an unexpected new challenge: when there is a dependency graph of ``pkg_a`` that depends on ``pkg_b`` and we are using the ``package_revision_mode`` and building the whole graph from source with ``--build=pkg_b``, the ``package_id`` of ``pkg_a`` cannot be computed. As the ``package_revision`` of ``pkg_b`` doesn’t exist yet, because it is computed from the binaries yet to be built, the information to consumer ``pkg_a`` ``package_id`` doesn’t exist yet. So at the time of computing the dependency graph, it is even impossible to know what actions will be necessary for ``pkg_a``, if it will be possible to download it, or if we will need to build it from source, etc. Only after the ``pkg_b`` has been built from source will we have that information, and then it will be necessary to re-compute downstream in the dependency graph. Because this effect is transitive, all the packages depending directly or indirectly on ``pkg_a`` and ``pkg_b`` will result in the same "unknown" package id.

At the core of this unexpected scenario there is a hidden pathological situation. In theory, every package binary with a given ``package_id`` should only have 1 ``package_revision``. The only situation when more than one ``package_revision`` happens should be a system error in which a binary is unnecessarily being re-built from sources, from the exact same sources (because if something is changed in source, a new recipe revision is produced, with completely new binaries), and with the exact same package_id, which means that all the input configuration, settings, options, and all dependencies are exactly the same. That should never happen in a properly working system, and the only possible explanation is that something is being changed in the system without any representation at all in source code. This is known to be an anti-pattern in modern SW development, and, consequently, Conan should not need to struggle trying to handle this challenging case.


## Detailed Design

Conan will remove the ``package_revision_mode`` from the available modes both in the configuration default and in-recipe. The maximum reference that can be used in consumer packages will be ``pkg/version@user/channel#recipe_revision:package_id`` which strongly guarantees that any upstream change in source code and/or in configuration will require and force the build from sources of new binaries of such dependent packages.


## Migration plan

Users relying on this feature in Conan 1.X should update their processes and use a package_id mode that doesn’t include the "package_revision". There are 2 different strategies for this:

Using a mode that includes the ``recipe_revision``, when things are to be re-built, then it is enough to do a simple change in either the recipe or the source code. For recipes that use ``scm``, a new commit will also do it. The idea is that every build from source is always driven by something that is reflected in the history of the source repo.
If the need to rebuild binaries from exactly the same source, and exactly the same configuration happens because of some reason outside of source control and Conan, then a whole forced rebuild of the packages will be enough. Not requiring to rely on the ``package_revision_mode`` to tell Conan that packages need to be rebuilt, because that should always result in rebuilding everything anyway. So a ``--build`` argument to force rebuild everything will achieve the same effect.
