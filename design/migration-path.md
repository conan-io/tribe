# Proposal: Migration to Conan 2.0 general guidelines

| **Status**        | **Accepted**                                      |
|:------------------|:--------------------------------------------------|
| **RFC #**         | [013](https://github.com/conan-io/tribe/pull/13)  |
| **Submitted**     | 2020-11-25                                        |
| **Tribe votes**   | :thumbsup: (47) :thumbsdown: (3) :eyes: (0)       |

---

## Summary
This document defines some general principles about the future upgrade to Conan 2.0 path that users will need to do.
In summary:
- Conan recipes will implement a subset of syntax and features in Conan 1.X that is fully operational and can run in Conan 2.0 without requiring changes in conanfiles.
- The command line in Conan 2.0 will change, in a non 1.X backwards compatible way. Conan 1.X command line syntax will be adapted as much as possible to the Conan 2.0 new one, but it might not be completely possible for some cases.
- Behavior will change, in a non 1.X backwards compatible way.
- Configuration files might change, in a non 1.X backwards compatible way, but trying to reduce the changes to a minimum.
- When Conan 2.0 is released, it will become the active series, and Conan 1.X will only get important bug fixes.

Other aspects not commented here, for example, if the Conan cache will be compatible, are unknown at this moment, so they are not included. These things might be discussed later if necessary.


## Motivation
Conan 2.0 is a great opportunity to get rid of some existing legacy, outdated or not optimal defaults, and deprecated practices
and features. The only way to achieve this is by introducing some breaking behavior. On the other hand, it is necessary to
define an upgrade path that is as smooth as possible, with a special focus on avoiding the split of the ecosystem like it happened
with Python 2 and 3.


## Proposal

### Conan recipe compatible syntax 1.X => 2.0

This doesn't mean that all Conan 1.X syntax and features will work in Conan 2.0. Many things will be deprecated and removed.
It means that Conan will provide the features that will be defined as the canonical and recommended ones in Conan 2.0 in 1.X.
These features will be declared stable in 1.X when there is enough usage evidence, to allow companies to safely adopt them.
As an example (and this shouldn't be discussed in this proposal, it is just an example), the current ``CMake`` build helper will be
deprecated, and using the new ``CMakeToolchain()`` plus the new build helper that uses the generated toolchain file will be the
feature that will work in Conan 1.X and will continue to work in Conan 2.0.

Implementing this forward compatibility is critical for the upgrade. Conan 2.0 cannot require in recipes any new feature
that is not already supported in Conan 1.X. Users, (and that includes **ConanCenter** central repo) will need to gradually adapt
and prepare their recipes during 1.X, adopting the features that will keep working in Conan 2.0.

### Conan command line will change in 2.0

This is the only possibility. There is a lot of evidence that some commands are not the most intuitive (e.g. the ``search`` should
by default search in remotes, not in the cache. Again, just an example, not something to discuss here), and the only possible way
forward is to change the command line syntax and behavior. Contrary to the Conan recipes, here it is not possible to implement
a "compatible forward" transition, and users will need to upgrade their scripts using Conan to the new Conan 2.0 syntax.

Some new commands have already started to be designed with some new guidelines for the Conan 2.0 CLI. For example, the ``conan lock``
command uses always the full path to the files, like ``conan lock create conanfile.py``, because the Conan 2.0 will be more explicit,
to avoid some confusing ambiguities of the current CLI syntax. But the other commands will need to change, and in most cases, it is
impossible to do it in a non-breaking way, for example ``conan install path/conanfile.py`` would be incorrectly processed as a ``pkg_name/version``.

The Conan 2.0 new command line will be designed and implemented from scratch, taking the existing 1.X one as a reference, but without being constrained by it, not trying to be backwards compatible. Close to the Conan 2.0
release, and once the Conan 2.0 CLI is matured enough, an effort will be done to backport it as much as possible to Conan 1.X, and provide an upgrade path that minimizes breaking changes.

### Behavior will change in 2.0

The fact that recipes will be forward compatible doesn't imply that the behavior will be the same. Some things might change,
for example (again, as an example, not to be discussed here), the default ``package_id_mode`` might change, and then it might become necessary
building new binaries. Or the ``build_requires`` recipes will always be fetched, even if they are not needed for building from source,
making the installs a bit less efficient.

There are a few Conan defaults that made sense a few years ago, but not anymore nowadays, like the default ``settings.libcxx=libstdc++``, even
for gcc>5 compilers. Conan will change this and use ``libstdc++11`` whenever available, but this is breaking behavior.

### Configuration files might change in 2.0

It is very likely that most part of the current configuration files will keep the same, like ``profiles`` or ``settings.yml``.
But there are other files that might have changes, like ``conan.conf``, which currently contains a lot of variables that don't
belong there, but to profiles. The way that remotes are defined in the ``remotes.txt`` file might also change.
Following the above idea, whenever possible, alternatives will be available in 1.X, but in this case it cannot be fully guaranteed that
the same ``conan.conf`` file will work both in 1.X and 2.0 without breaking.

As the ``conan config install`` functionality can easily install and manage new configuration files, this shouldn't be an issue at all.


### When Conan 2.0 is released, it will become the active one

The Conan community needs to avoid the Python2-3 split, and dividing resources and continue development on 1.X once 2.0 is released
is not only very inefficient, but almost unfeasible.

Once Conan 2.0 is out, Conan 1.X will get strictly bug fixes and only the necessary things to help in the transition to Conan 2.0.
It will not get any new feature, or backport Conan 2.0 functionalities.


## Detailed Design
There will be a detailed document summarizing all the deprecations and features to adopt in Conan 1.X to allow the upgrade.


## Open issues


## Future Extensions
