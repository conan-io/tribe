# Proposal: Dependency graph will always be computed completely, including build-requires.

| **Status**        |                                                   |
|:------------------|:--------------------------------------------------|
| **RFC #**         | [020](https://github.com/conan-io/tribe/pull/20)  |
| **Submitted**     | 2021-03-15                                        |
| **Tribe votes**   |                                                   |

## Summary

Whenever the Conan dependency graph is evaluated and computed, all the transitive dependencies will be computed, including ``build_requires`` and their transitive dependencies. This includes many commands such as ``conan create``, ``conan install``, ``conan info``, and ``conan lock`` among others.

Only the recipes will be required to perform the evaluation. Once evaluated, only the binaries which are needed will be downloaded.


## Motivation

At the moment, the ``build_requires`` are only computed when a package is marked as “necessary to build”. Otherwise those dependencies are not added to the graph at all. They are not computed and they are not fetched. The current algorithm to compute dependency graphs is recursive, and can be summarized as follows:

1. Starting from the initial conanfile, compute the dependencies and transitive dependencies of regular ``requires``. We could have at this stage a ``conanfile.txt``->``poco/1.9.4``->``zlib/1.2.11``.
2. Compute which packages need to be built, based on the current existing binaries and the ``--build`` policies
3. For those packages that need to be built, compute their ``build_requires``, let's say that we need to build ``poco/1.9.4`` from sources, and it has a ``build_requires`` that is ``cmake/3.14``.
4. Expand the graph with the ``build_requires`` and its transitive ``requires``. In this example, the algorithm would add ``poco/1.9.4``-(build_requires)->``cmake/3.14``.
5. Go to 2, until no more packages are marked as "necessary to build"
6. Install the binary packages, downloading pre-compiled binaries, finding them already in the cache or building them from sources


This approach has some disadvantages:

- Build-requires can never affect the ``package_id``. Even if this is a user's request, it is impossible by its own definition. Unless a package is marked as “necessary to build”, the ``build_requires`` are not computed. But to know if a package needs to be built or not, first it is necessary to compute its ``package_id``. This chicken-and-egg problem makes impossible that build_requires could affect their consumers ``package_ids``
- It is more complicated to manage and orchestrate projects builds, for example in CI. Unless careful management of ``--build`` policy is done, it is easy that build_requires could be missing in lockfiles for example.
- It makes the Conan codebase much more complicated, and the treatment of dependencies more heterogeneous, which makes thorough and flexible management of dependencies more difficult. The extra maintenance slows down development.


## Proposal

When the dependency graph is computed, the whole dependency graph will be computed. The algorithm could be outlined as:

1. Starting from the initial conanfile, compute the dependencies and transitive dependencies of ``requires`` and ``build_requires``. We could have at this stage a ``conanfile.txt``->``poco/1.9.4``->``zlib/1.2.11``, and also ``poco/1.9.4``-(build-require)->``cmake/3.14``
2. Compute which packages need to be built, based on the current existing binaries and the the ``--build`` policies
3. Install the binary packages, downloading pre-compiled binaries, finding them already in the cache or building them from sources


## Detailed Design

This approach would solve the above issues. It will have some other potential issues:

- It might be slightly less efficient, as it might need to resolve some extra recipes as ``cmake/3.14`` that wouldn’t be strictly necessary otherwise. But only the recipes will be necessary, downloading and unzipping the potentially heavy binary packages can still be avoided, so the impact on the overall time might be negligible. In practice, it could suppose and average penalty of around 1-2% of the installation of a graph with existing binaries (much less if something is built from sources)
- It requires the ``build_require`` recipes to be accessible and available in different stages, for example, if recipes are copied from one server to another one, it is necessary to copy too those ``build_requires``, if later we plan to do a ``conan install``. Note that if the ``build_require`` is injected via the profile, it would be enough to not pass it in the profile, and it will not be necessary (but in this case the binaries ``package_id`` should be independent of the ``build_requires``). Also note that in many cases this is not necessarily a disadvantage, but desired behavior, as it allows full build reproducibility from the new server too.
- It can resolve to invalid binaries in some configurations, but this can be a non-fatal condition. For example, when cross-building. Let's say that we have a cross-compiler "mygcc-arm/1.2" that works in Windows and can cross-compile for Linux arm architecture. If this is a ``build_require``, in Windows we could use a ``conan create . -pr:h=LinuxArm -pr:b=Windowsx86`` with the build and host profiles. Once the binaries are built, we could go to a Linux box and do ``conan install … -pr:h=LinuxArm``, but the binary for the cross-compiler “mygcc-arm/1.2” will not exist for that platform, again this is not necessarily an error.  The binary can simply be marked as INVALID by the ``validate()`` method, and unless we tried to build, it would still be possible to install and use the desired binaries.

As always, this is a tradeoff, and these possible issues seem small compared to the advantages. In any case, please note that it is impossible to have all advantages without the disadvantages, at least without an extraordinary complexity that will be harmful for the project and future development and maintenance. So we cannot have at the same time the possibility that ``build_requires`` affect the consumers ``package_ids`` and those ``build_requires`` not being retrieved always to compute the graph. Please take it into consideration when discussing this proposal.


## Migration plan

This proposal will not require changing the recipes. The current ``build_requires`` definition will still be valid. Also, the default effect on the ``package_id`` will be "no-effect", as it is today, and users that want this behavior will have a mechanism to opt-in this behavior via configuration or recipe syntax. The only possible incompatibility would be recipes that raise ``ConanInvalidConfiguration`` for cross-build scenarios in other places rather than the ``validate()`` method.
