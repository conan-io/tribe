# Proposal: Use always the build_profile and the build and host contexts.


| **Status**        |                                                   |
|:------------------|:--------------------------------------------------|
| **RFC #**         | [022](https://github.com/conan-io/tribe/pull/22)  |
| **Submitted**     | 2021-05-06                                        |
| **Tribe votes**   |                                                   |



## Summary

Use always, for all install/create commands both the "build" and the "host" profiles,
being the "build" one the configuration for the current building machine, and "host" the configuration
for the system in which the final built application will run.

If one or both of the build and host profiles is not specified, the "default" one will be used.


## Motivation

Having build requires like ``build_requires = "cmake/3.15.0"`` in the legacy model has been problematic, specially for cross-building.
Because by default (in Conan 1.X) those build requires live in the "host" context, that is the same context as the final built application
will run. So if we are trying to cross-compile from Windows to Linux and we have a ``build_requires = "cmake/3.15.0"``, the ``cmake/3.15.0``
package will be downloaded for ``os=Linux``.

Recipe creators have been trying to workaround this with the ``os_build`` and ``arch_build`` settings, but this requires more complexity into
the recipes. Furthermore the model is broken for packages that can live in both "build" and "host" contexts, like ``protobuf``, which contains
both the library to link with (host context), and the "protoc" application to be used at build time to generate code (build context).

The ``--profile:host=mylinuxprofile --profile:build=mywindowsprofile`` was introduced long time ago, and has proved to be a better approach.
It has been opt-in in Conan 1.X, and the way to use this feature is to explicitly use ``--profile:build`` argument.

## Proposal

Use always, for all install commands both profiles, the host and the build one.
Even when they are not defined in the command line, they will always have a value, using the "default" auto detected profile from the cache if necessary.

These commands will be equivalent and do the same:

```bash
$ conan install .
$ conan install . --profile=default
$ conan install . --profile:host=default --profile:build=default
```

The ``os_build``, ``arch_build`` and ``os_target``, ``arch_target`` settings will be removed from ``settings.yml``.

By default ``build_requires`` will live in the "build" context (unless ``force_host_context=True`` or something equivalent is defined)


## Detailed Design

While it seems there is enough evidence that the new model is better, there are still some other related issues to be improved:

- How the information from packages in the build and host contexts is consumed depends on the generators used.
The legacy 1.X generators might be lacking some functionality, but the new generators and build system integrations in ``conan.tools.xxx``
space will use this new model with build and host profiles. There are ongoing efforts to augment their functionality, for example
``CMakeDeps`` might be able to generate *xxx-config.cmake* scripts for packages in the build context (like ``doxygen``, ``protobuf``, etc),
that could be used for build automation within CMake.

- Also, how the environment is managed, depends on the build/host contexts. In the new mode, ``self.env_info`` from dependencies in the "host"
context will not automatically be injected during the build, but only the environment from "build_requires".
This is also ongoing work in the new ``conan.tools.env`` tools.


## Migration plan

Users can start using this mode by defining ``--profile:build=xxx`` in Conan 1.X, which might be redundant in some cases in Conan 2.0, but will
continue to work.
