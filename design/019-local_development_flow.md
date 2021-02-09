# Proposal: New local development flow: “conan install” + use your build system

| **Status**        |                                                   |
|:------------------|:--------------------------------------------------|
| **RFC #**         | [019](https://github.com/conan-io/tribe/pull/19)  |
| **Submitted**     | 2021-02-05                                        |
| **Tribe votes**   |                                                   |

---

## Summary

The so called Conan “local flow”, composed by the commands ``source``, ``build``, ``package`` will be simplified to the following:

- The main local development flow will be just ``conan install`` plus using the native build tools. The already existing ``generate()`` method, together with the ongoing work in the ``conan.tools.xxx`` new helpers like ``CMakeDeps`` and ``CMakeToolchain`` should generate all the files and information necessary to build, without requiring Conan or the ``conan build`` command.
- Remove the ``conan package`` command, its functionality is fully covered by the ``conan export-pkg`` command
- The ``conan build`` command might fully disappear if the above doesn’t have any exception. If there are some flows that cannot be fully fulfilled with a pure generator approach, a local ``conan build`` might be useful. If that is the case, the command will be complete, computing a full dependency graph that doesn’t rely on the previously saved state.
- The locally generated files ``conanbuildinfo.txt``, ``conaninfo.txt`` and ``graph_info.json`` will no longer be necessary and will disappear and be removed.


## Motivation

The local ``conan package`` command was intended for testing and debugging the package creation and specifically the conanfile.py ``package()`` method.
It performs such process locally, packaging to a user provided "mypackage" folder. This can implement only a partial test, it cannot do things such as computing the real final ``package_id`` that the package will have, and doesn't make the result available for consumers to test it. Moreover, it is also useless for possible local flows as using “editable” packages or “workspaces”, because those will use the “build” artifacts, not the package ones in that local folder.
On the other hand, the ``conan export-pkg`` command do exactly that, performs a full evaluation of the graph, the ``package_id``, and do a real ``package()`` directly in the cache, making the package immediately available for consumption. The process is almost equally fast, and the result is a folder anyway, that can be inspected manually for debugging. The only difference is the location of the folder, which will be in the local cache (the location in the cache might be varying if the recipe has changes causing a new recipe revision). Then the ``conan package`` command doesn't provide any value that is not already implemented in ``conan export-pkg``. It is important to remark that ``conan package`` was never capable of deploying to a local folder. It is just a pure local command that requires the built artifacts to be already in the user folder.

The local ``conan build`` command was intended to easily call the current recipe ``build()`` method, directly in the user folder. The main problem is that the ``build()`` method needs some state: the settings, the options. So ``conan build`` started to use a local ``conaninfo.txt`` file that was generated in a previous ``conan install`` command. Also, it was necessary to have the ``deps_cpp_info`` information, so that was serialized to a ``conanbuildinfo.txt`` also in a previous ``conan install`` command. Finally, the name, version, user, channel of the recipe was also necessary, and that is recovered from a serialized ``graph_info.json`` file, from the previous ``conan install``. All the state that might be necessary for a build needs to be recovered from a serialized representation from the previous ``conan install``. This makes this approach fragile, as any missing information or small bug in serialization/deserialization causes errors and problems in that local build. It is also frequently confusing for users, from a UX perspective, that they cannot ``conan build -s build_type=Debug``, because build can only use that information from a previous saved state.

This ``conan build`` command was useful because the integration with build systems relied a lot on the helpers called in the recipe ``build()`` method. With the new ``generate()`` method and the toolchain, the goal is to achieve a very transparent integration with the build systems, in which the user can more easily call the build system and achieve exactly the same build as will happen in the Conan cache if the package is created with ``conan create``.


## Proposal

- To remove the ``conan package`` command and propose to use ``conan export-pkg`` to test/debug the ``package()`` functionality if necessary.
- Keep working, providing in ``conan.tools.xxxx`` new tools and toolchains a full working set of helpers to use in the ``generate()`` method, that operate creating files for the user convenience, so users can build directly with their native tools, instead of relying on the ``conan build` command.
- One of two scenarios could happen: If the above works always, then the ``conan build`` command might become completely useless and will be removed. If on the contrary, there are still some use cases that cannot fit the ``generate()`` approach, Conan 2.0 will make the ``conan build`` complete, not using any saved state representation from a previous install. The 3 files ``conaninfo.txt``, ``conanbuildinfo.txt`` and ``graph_info.json`` will not be generated at all locally. The ``conan build`` command will accept the same arguments as ``conan install``, settings, options, profiles, lockfiles and will compute the graph of dependencies and provide the exact same state to the ``build()`` method as if the package was being created in the cache.

NOTE: There are a couple of side use cases, like ``source`` and ``imports`` functionality, that can be relevant but are not as core as the one presented here. Some other proposals can happen regarding these other elements, let's leave those out of this proposal and discussion.


## Detailed Design

In the case that the ``conan build`` command is maintained, its only minor inconvenience could be that it would be slower than the current version, as it will evaluate the whole graph of dependencies. This shouldn't be a blocker for some reasons:

- For small and medium size projects, this is not relevant at all. For large projects, with hundreds of packages in the dependency graph, it might take a few seconds.
- The ``conan build`` command will not be necessary in most cases. There is a lot of effort ongoing on the new toolchains approach to integrate build systems to have native builds, that do not require Conan to build locally, and thus can use the IDE and other native tools, providing a better developer experience.
- The time used to compute the whole graph, for large graphs, is still in the order of seconds, which can be typically small compared with actual build times.
- Conan 2.0 will implement a new graph model. There are some ideas to improve the graph resolution performance, it is possible that the time to evaluate dependency graphs might be reduced.


``conan export-pkg`` will be simplified, as it will always execute the ``package()`` method. For existing external binaries, it could be just a ``self.copy(“*”)``, if the artifacts are already arranged in the final layout. But as the local “packaged” state is no longer a use case, the functionality of ``export-pkg`` that was skipping the ``package()`` method is no longer necessary.


## Migration notes
These commands are not frequently used in automated or CI flows, mostly developer side commands. No migration issues expected.
