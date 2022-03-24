# Proposal: Visual Studio minimum version and settings support


| **Status**        |                                                   |
|:------------------|:--------------------------------------------------|
| **RFC #**         | [031](https://github.com/conan-io/tribe/pull/32)  |
| **Submitted**     | 2022-02-11                                        |
| **Tribe votes**   |                                                   |


## Summary

Conan 2.0 will implement Visual Studio compiler support via the ``msvc`` compiler setting, that
will use the compiler version instead of the IDE version. The legacy ``Visual Studio`` compiler
setting will be dropped.

The supported versions will be:
- Visual Studio 2017 will have tested (in Conan CI) support, both for the ``MSBuild`` and ``CMake`` integrations.
- Visual Studio 2015 will have community-tested support both for ``MSBuild`` and ``CMake`` integrations.
- Visual Studio 2012 will have community-tested support for ``CMake`` integrations.
This support will be stable and not dropped. The necessary features to support toolsets like ``v141_xp`` will be added to ``msvc`` settings.


## Motivation

One of the first tribe proposals, the one proposing [Visual Studio 2017](https://github.com/conan-io/tribe/pull/5) as the minimum supported version in Conan 2.0 was dropped because of the
requests to model the Conan settings after the compiler version and not the IDE version, as a more
important action than defining the minimum supported version.

The new ``msvc`` compiler setting has already been introduced as experimental in Conan 1.X and after
a few iterations, it seems it is a valid approach, and it could be moved forward. Now it seems based
on the accumulated experience, we could define some base Visual Studio support in Conan 2.0.

This initial proposal was limited to 2017 tested support and 2015 community support via CMake, but has been expanded after some
extra testing in https://github.com/conan-io/conan/pull/10808, and validation by some users.


## Detailed Design

The Conan built-in settings define the new compiler ``msvc`` as:

```yaml
    msvc:
        version: [170, 180, 190, 191, 192, 193]
        update: [None, 0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
        runtime: [static, dynamic]
        runtime_type: [Debug, Release]
        cppstd: [98, 14, 17, 20, 23]
        toolset: [None, v110_xp, v120_xp, v140_xp, v141_xp]
```

The ``version`` fields matches the first 3 digits of the [msvc compiler version](https://en.wikipedia.org/wiki/Microsoft_Visual_C%2B%2B):
- IDE versions 14.0 (VS 2015) map to compiler version 1900
- IDE versions 14.1-14.16 (VS 2017) map to compiler version 1910-1916
- IDE versions 14.20-14.29 (VS 2019) map to compiler version 1920-1929

So as the ``version`` field, the matching digits that are stable for an IDE version are used. 
This defines the "base" binary compatibility for Conan, the one used by default in ConanCenter:
the binaries compiled with the same IDE version are compatible by default.

For users wanting to opt-in into more granular binary compatibility, they can use the ``update``
version number, which correspond to the last digit in the ``msvc`` compiler version.

Also note that the runtime is now defined as "static" or "dynamic", and not as the raw MT/MD flags, 
accordingly the ``runtime_type`` specifies the Debug/Release variant of this same runtime. Also, the
``cppstd`` field is mandatory, even if it is the compiler default ``14``, it has to be specified in profiles.

## Implementation details

- The latest Conan 1.X release already implements ``msvc`` support for the new build system integrations.
- The previous ``Visual Studio`` setting will be removed in Conan 2.0

The integration of LLVM/Clang compilers via VS toolset might be provided by the ``compiler=clang`` setting and adding more information to that
compiler, because the actual compiler is clang, not Microsoft cl, following the same rationale applied in the compiler version for ``msvc``, the
important bit is the compiler not the IDE and not the build system.


## Migration plan

Conan 1.X already implements a path forward. There is default binary compatibility that maps betweeen the ``Visual Studio`` and ``msvc`` compiler settings. Users moving to ``msvc`` settings
shouldn't need to rebuild immediately all binaries, but it can be incremental.

Only the new build system integrations that will exist in Conan 2.0 fully implement ``msvc`` support, so this assumes moving to the new integrations is necessary. That means that the path to
upgrade should be first, move to the new build system integrations, second move to the ``msvc``
settings.
