# Proposal: Define Package Types


| **Status**        | **Accepted**                                      |
|:------------------|:--------------------------------------------------|
| **RFC #**         | [027](https://github.com/conan-io/tribe/pull/27)  |
| **Submitted**     | 2021-07-06                                        |
| **Tribe votes**   | :thumbsup: (28) :thumbsdown: (0) :eyes: (3)       |


## Summary

Introduce the optional concept ``package_type`` in recipes, to specify which kind of artifacts it contains. Possible ``package_types`` will be "shared-library", "header-library", "application", "library", etc.

The ``package_type`` is not a mandatory field, for many cases it will not be necessary to define it in recipes. The ``package_type`` is proposed as an optional way to complete information in scenarios where other existing mechanisms are not enough, for example, representing the information that a package contains a header-only library is not possible with existing Conan 1.X mechanisms, as options. The goal of the ``package_type`` is not to replace or modify how existing options like ``shared`` work, but to complete their information when they cannot do it on their own.

The ``package_type`` does not intend to be an exact, complete and accurate representation of every artifact that is contained in the package, in exactly the same way the current options ``shared=[True, False]``, existing in multitude of Conan packages apply only to the library/libraries contained in that package, but not to the executables that are also contained within that same package. For a packaged library that is always shared (and thus the option model does not work) a class attribute ``package_type = "shared-library"`` will be correct, even if the package contains other things like executables.

The fact that a package can define its package type as ``shared-library``, for example, does not exclude the possibility to use its packaged executables. The main mechanism to define the relationship between packages is the "Requirement traits" described in the proposal https://github.com/conan-io/tribe/pull/26. The ``package_type`` can just help that requirements model providing some information about the default role of a package in a typical ``requires`` relationship, for example, when another package is requiring ``zlib/1.2.11`` Conan is already assuming that this is a library to be consumed and linked. Build-requirements, deploys, installs, virtualenv can still be used to run executables from packages, irrespective of their ``package_type``, by defining the correct "requirement traits".


## Motivation

Conan's main declaration of dependencies is the ``requires`` relationship. While it works well to specify package to package generic dependencies (specify versions, handle version conflicts, etc), it fails to represent the specific relations that happen between components like C and C++ libraries. A library (contained in a Conan package) could be a static or shared library, and how it is used, consumed, propagated and lately deployed or not into production, is affected by the type of the library.

While some attributes could be added and specified in the ``requires`` (see other proposal https://github.com/conan-io/tribe/pull/26), the type of the package/library is not one of them, as it would require to define something as ``requires_static`` or ``requires(static=True)``, directly in recipes, which could be really inconvenient and rigid, if not directly easily leading to configuration conflicts (some packages requesting a static lib while others request a shared lib).

Packages already contain some implicit information regarding their type, for example, if a package declares the ``shared`` option, it could be deduced that it is a library, that it can be either shared or static. The ``header_only`` option would do something similar. But this information is also not enough, there might be packages that for some reason are always a static or a shared library: specifying an option with a single value for it is almost an anti-pattern and an unnecessary source of possible issues (as someone specifying ``*:shared=xxx`` downstream, which won’t match the only valid option, will raise an error). The case of a package that is packaging a "header only " library  is paradigmatic, there is no way to convey this information with current Conan mechanisms like options.

The type of library also directly impacts binary compatibility and the ``package_id`` computation. Currently in Conan 1.X, the default ``semver`` mode, or even the other ones, like ``minor_mode`` or ``recipe_revision_mode``, always apply the same effect on the consumers ``package_id``, irrespective of the dependency type. But it is known that if a shared library is linking a static library, the second binary code will be embedded in the shared one, so it seems that a more restrictive mode like ``recipe_revision_mode`` should apply: for every change in the dependency, the downstream consumer should have a new ``package_id`` to guarantee that it is rebuilt to embed the static dependency. But just a change in the dependency ``option shared`` to shared library, dramatically changes it, and something like ``semver_mode`` could be appropriate, as changes in the dependency will only affect the consumer binary if those changes are in the public headers and API of the dependency (and would seem appropriate to do a version bump for those changes). Then, internal changes to the shared dependency do not need a new ``package_id`` or a rebuild in the consumer. Same happens with a static-static dependency. Having the package type information would help to define better binary models that take into account this variability.

Something similar happens with the information propagation and visibility. It is undesirable to have visibility over the transitive dependencies headers, as they will introduce some coupling in our code that is not modeled in the dependency graph, and not subject to direct versioning. As a general rule, if some package is explicitly including with ``#include`` some headers of the dependencies, that package should have a ``requires`` to it, as to define the version of that dependency. When a static dependency transitively requires another static dependency, the final application consumer should get both in the linking step, but if a shared dependency depends on a static library, the final application consumer should get only the shared dependency in the linking step.


## Detailed Design

Package recipes will contain a new attribute that defines the package type:

```python
class Pkg(ConanFile):
    settings = "os"
    package_type = "shared-library"
```

It is also possible to define such attribute in the ``configure()`` method:

```python
class Pkg(ConanFile):
    def configure(self):
        if something:
            self.package_type = "shared-library"
        else:
            self.package_type = "application"
```

The initial proposal is to define the following package types:

- "static-library": a regular static library (.lib, .a) with its public headers. To be defined only when a packaged library is always static (no "shared" option exists).
- "shared-library": a regular shared library (.so, .dll, .dylib) with its public headers, and in the case of Windows, the typical small .lib linkage library. To be defined only when a packaged library is always shared (no "shared" option exists).
- "header-library": a library containing exclusively headers, but not binary compiled artifacts. To be defined only when a packaged library is always shared (no "shared" option exists).
- "library" : This is an abstract type, this type will be resolved to any of the 3 above cases based on options. The ``shared`` option must exist and have a value, otherwise, the recipe will fail. The optional ``header_only`` option would be used to define if it is a "header-library". This type is not necessary to be defined, but if it is defined, it will enforce the existence of a ``shared``.
- "application": Executable artifacts, .exe, binaries to run. It will not contain headers, static libraries, or shared libraries to be linked (might contain shared libraries used by the executable, but exclusively for the executable). Not necessary to define it, but it might be informative for recipe readers to realize that such "conanfile.py" is creating an application and not a library.
- "python-require": This is a pure python recipe, it actually doesn’t contain any package binary or artifact at all.
- "assets": This package contains some assets, like images, sound files, videos, fonts, configuration files, etc. They are not C/C++ headers, not libraries to link or applications to run.
- "build-scripts": This package contain some files useful for building. For example, it could be a set of ".cmake" files with utilities that can be called by consumers of this package.
- "unknown": If the type is not defined explicitly, or cannot be deduced from options

This is just the initial proposal, please report on any other potential package types that you are already managing (better not to speculate, but build on top of actual use cases) and would benefit from an explicit model.

This proposal is only about the package types, defined as their common or default role in dependencies. While package types could help somehow to the dependencies information propagation, as described above, that is mostly controlled by the "requirements traits" model. Lets focus this proposal on these possible package roles in dependencies.


## Implementation details

The package type will be an enumerated type like:

```python
class PackageType(Enum):
    LIBRARY = "library"
    HEADER = "header-library"
    APP = "application"
    PYTHON = "python-require"
    UNKNOWN = "unknown"
```

To allow some type and value safety, but in general recipes should use the string format, for simplicity, while internal code might use the enum more frequently.

Package types of dependencies will be accessible, via the ``self.dependencies``, like:

```python
def generate(self):
     dep = self.dependencies["mydep"]
     dep_type = dep.package_type
     if dep_type == "shared-library":
            ...
```

## Migration plan

As the attribute has no effect in Conan 1.X it is possible to start defining their values directly in Conan 1.X recipes, to have them ready for the new behavior that Conan 2.0 will enable when it sees this ``package_type``. But this is not necessary for most cases. The recommendation would be to add it for "header-only" libraries that do not declare options "shared" and for compiled libraries that for some reason they cannot implement either the static or the shared variant, and thus the "shared" option is not a good solution.
