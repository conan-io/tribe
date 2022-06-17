# Proposal: Conan 2.0 requirements effect in PackageID

| **Status**        |                                                   |
|:------------------|:--------------------------------------------------|
| **RFC #**         | [036](https://github.com/conan-io/tribe/pull/36)  |
| **Submitted**     | 2022-06-17                                        |
| **Tribe votes**   |                                                   |


## Summary

This proposal is about the effect that the dependencies have over the current package ``package_id``, but not about other factors, such as the package settings, options, configuration or other type of binary compatibility resulting from other factors other than dependencies.

Conan 2.0 proposes a new default ``package_id`` that will be dynamic, based on two main cases:

- When an application is linking a static library, or a shared library is linking a static library, or any library or application is linking a header-only library as an implementation detail, the resulting artifacts will be fully embedding/inlining the dependency artifacts, and they require in the general case to be re-built from sources when something in the dependency changes. This will be the "embed" case, and by default will use a "full_mode" policy, which means that the full dependency reference ``pkg/version@user/channnel#recipe_revision:package_id`` will be used and factored in the consumer ``package_id``. Recall that ``package_revision`` can no longer be part of the ``package_id`` as removed in proposal https://github.com/conan-io/tribe/pull/30
- When a static library is depending on a static library, or a header-only library is depending on another library, the resulting artifacts in general do not embed/inline the dependency artifacts, and they don’t need to be rebuilt in many cases even if the dependency changes. For example, doing a patch in an implementation file .cpp of a static library does not require another static library that depends on it to be recompiled. This will be the "non embed" case and by default will use a "minor_mode" policy, which means requiring a rebuild if the major or minor of the dependency changes, but not requiring it if the patch of the dependency changes.

Conan will be using the ``package_type`` information to define which of the above scenarios apply.

Conan 2.0 will allow ``tool_requires`` (or any other ``requires`` with trait ``build=True``) to affect the ``package_id`` of their consumers. By default it will be effect will be ``None``, but users will be able to define it, globally via ``global.conf`` configuration, or per recipe in the ``requires(...., build=True, package_id_mode=xxxx)`` trait (being ``xxx`` any valid mode like ``minor_mode``, ``patch_mode``, ``full_mode``, etc)

Conan 2.0 ``package_id`` will not depend by default on the dependencies’ ``options``. Their influence should be represented in most cases by the scenarios above, and the ``full_mode`` taking into account the dependency ``package_id`` into the consumer ``package_id``.


## Motivation

When Conan started, and even at the moment of stabilizing Conan 1.0, a few years ago, it seemed that "semver" would be a good default for defining the binary compatibility and when it is necessary to re-build something from source. As it quickly became evident, this default was not good enough for many common cases that happen when compiling native binaries, executables, shared libraries, etc, and when the C and C++ textual inclusion model for headers is processed.

Many common scenarios, like a shared library linking a static library that would become part of its binary, were difficult to model. One of the most common abuses in Conan 1.X of the ``build_requires`` feature is using it to try to represent this case.

Beyond the importance of having different binaries for different situations, and not "overwriting" binaries that should have different ``package_id``s, it also became evident that a more accurate modeling for computing "what needs to be rebuilt and what not" was necessary. Because the use cases for Conan have been growing in size, and there are many teams with large dependency graphs, and doing CI at scale with such graphs can be challenging, and become inefficient if not modeled correctly. A better ``package_id``, together with easier to use ``lockfiles`` was necessary for enabling such CI at scale.

Other feedback along 1.X versions also showed that for some teams it is also important to take into account the information from ``build_requires`` into the consumers ``package_ids``, because some tools new versions could indeed produce different binaries. Factoring in the influence of ``build_requires`` in Conan 1.X was impossible, a chicken-and-egg problem, because ``build_requires`` in 1.X are evaluated after the ``package_ids`` have been computed. In Conan 2.0, the dependency graph is always fully expanded, including ``build_requires``, and after that the ``package_ids`` are computed. That enabled the possibility of fulfilling that need.


## Detailed Design

Conan 2.0 will implement a new ``package_id`` default strategy which is dynamic, and can take different values for the versions of the dependencies, depending on the context and usage. The three main cases are:

- "embed" case: this mode will be used whenever some binary is "inlining" or "embedding" native binary code or resources of its dependency in its own artifacts. This happens when a shared library links a static library, when an executable links a static library, or when a translation unit of a library includes a header file from another library and inlines it. This mode typically means that when the dependency changes, the consumer has to be rebuilt to embed the changed dependency in its binary.
- "non embed" case: this mode happens when the resources and artifacts from the dependency are not inlined nor embedded in the current artifacts. An executable using a shared library or a static library "linking" (it actually doesn’t link, but this terminology can be used in build systems, to express a link time dependency) another static library. This mode means that in many situations when the interfaces of the dependency don't change, the binaries of the consumer don't need to be rebuilt when the binaries of the dependency change. If the interfaces of the dependency change, then bumping its version patch, minor, major, can define to its consumers if they need to rebuild (bumping the minor) or not (bumping the patch), or if they should expect breaking changes (bumping the major).
- "unknown" case: When Conan is not able to define one of the above scenarios. It will result in "unknown" and default to "semver_mode" (similar to what Conan 1.X was doing)

The identification of the scenario is based on the ``package_type`` and requirement traits. For example if the current package is ``package_type = "application"`` is doing a regular ``requires()`` of a dependency and such dependency is ``package_type = "static-library"``, then Conan will deduce that it is an "embed" case relationship.

The relationship of ``package_types`` and scenarios are:
- "embed" case:
   - Application or shared library, linking a static or header-only library
   - A static library linking a header-only library
- "non embed" case:
   - Application or shared library linking a shared library
   - Static library linking another static or a shared library
   - Header library depending on any other type of library

The above scenarios will use configurable "semver" modes, with the following defaults, which can be defined in ``global.conf``:

- ``core.package_id:default_unknown_mode``, default=``semver_mode``. This will produce different ``package_id`` when the major version of the dependency changes, that, is a dependency to ``zlib/1.2.11`` will be mapped to ``zlib/1.Y.Z`` (resulting in the same ``package_id`` for all different minors and patches. This mode should be minimal, as most packages should have a ``package_type`` defined, directly or indirectly via ``options`` like ``shared``.
- ``core.package_id:default_non_embed_mode``, default=``minor_mode``. This will produce different ``package_id`` when the minor version of the dependency changes, that, is a dependency to ``zlib/1.2.11`` will be mapped to ``zlib/1.2.Z`` (resulting in the same ``package_id`` for all different patches. The idea is that the following versioning is possible:
   - Bumping the "patch" version means an internal implementation change was done, but nothing that really affects the package interface (headers), so if the binary is not being embedded/inlined in the consumer, the consumer doesn’t need to rebuild
   - Bumping the "minor" version, means that some change was done in the public interface of the library. Maybe some headers that contain templates, or any other thing that could be inlined from headers in the consumers. This "minor" version bump means that the consumers need to be re-built, but in theory they won't break, and it will be possible to recompile them without changing the consumers source code.
   - Bumping the "major" version means that breaking changes were done in the public interface of the library that will break consumers. This change requires not only to rebuild consumers, but also possibly to change the source code of those consumers to match the new interface.
- ``core.package_id:default_embed_mode``, default=``full_mode``. This will produce different ``package_id`` when anything in the dependency version, down to the ``package_id`` of the dependency changes. The dependency will be mapped in the consumer ``package_id`` as ``pkg/version@user/channel#recipe_revision:package_id``. That means that if something in the recipe or dependency source code changes, the ``recipe_revision`` will change, and will produce a new consumer ``package_id``, requiring a rebuild. But also if the dependency ``package_id`` changes, maybe because an ``option`` that affects its binary is changed, then the consumer also needs to get a new ``package_id`` and a new binary that is compiled against that different dependency binary.


The above is for the regular ``requires``. But other types of requires can also affect the ``package_id``, with the following defaults:

- ``core.package_id:default_python_mode``, default=``minor_mode`` defines how ``python_requires`` will affect the ``package_id`` of the consumer using that ``python_require``. Note that the default is the - ``minor_mode``, because it can use the above versioning flow to represent "no need to build"=patch bump, "needs to rebuild, but won’t break code"=minor bump, "needs to rebuild, will likely break code"=major bump.
- ``core.package_id:default_build_mode``, default=``None``. By default the ``requires`` in the "build" context (``tool_requires``, ``build_requires``), do not affect their consumers ``package_id``. But with this configuration it can be done.


To summarize, things that can be configured (defined in ``global.conf``):
- ``core.package_id:default_embed_mode``
- ``core.package_id:default_non_embed_mode``
- ``core.package_id:default_unknown_mode``
- ``core.package_id:default_python_mode``
- ``core.package_id:default_build_mode``

Values that these configurations can take (given the full version is ``1.2.3``):
- ``full_mode``: including ``dep/1.2.3@user/channel#recipe_revision:package_id``
- ``major_mode``: include ``dep/1.Y.Z@user/channel``
- ``minor_mode``: include ``dep/1.2.Z@user/channel``
- ``patch_mode``:  include ``dep/1.2.3@user/channel``
- ``semver_mode``:  include ``dep/1.Y.Z@user/channel`` or ``dep/0.1.2@user/channel`` if major<1
- ``unrelated_mode``: empty, not influenced by it at all


Note that the above are general defaults. Every recipe can define the way they depend on other packages with the ``requires(...., package_id_mode=xxxx)`` trait.


Regarding the ``options``, by default the ``package_id`` will only use the ``options`` of the current package, but not the options of the dependencies, if the options of the dependencies can affect the binary, it should be covered by the ``full_mode`` of the "embed" case above. If that is still not enough, then users will be able to add the dependencies options into the package_id, in the recipe ``package_id()`` method.

NOTE regarding components: The ``package_id`` is a global per-binary package binary value. It doesn’t make sense to model it at the ``components`` level. If a package contains multiple components, like a mix of multiple libraries of different types (some components are static libraries, other components are shared libraries), it is up to the user to define the ``package_type`` that better represents the behavior of the package, and ``package_id_modes`` can always be defined at the ``requires`` level too if necessary.


## Implementation details

The configuration for defaults is global, and managed in ``global.conf``. The idea is that changing these defaults should be a team or company wide policy, and define it for most projects and teams. Otherwise, trying to use different default ``package_id`` modes, would very easily result in binaries constantly missing, and users frustrated. This is the reason why the configurations are ``core.package_id`` ones, and cannot be defined in profiles.

Also, very importantly, the new ``package_id`` will be computed directly hashing the contents of ``conaninfo.txt`` file included in every package binary. This way, understanding why one binary is different from another or why a binary would not satisfy some constraints will be easier, reading and comparing ``conaninfo.txt`` files.


## Migration plan

Conan 2.0 will always generate different and incompatible packages than 1.X. That means that it will be necessary to build new binaries for all recipes with Conan 2.0.

Conan 2.0 binaries will be able to coexist in the same server repositories as 1.X packages. As the ``package_ids`` are different, they will not collide. Following the agreed migration path, the recipes should be the same, and make them compatible, syntax wide with Conan 1.X and 2.0. Then different CI pipelines can build the packages for 1.X as usual, while in parallel binaries for 2.0 are produced.
