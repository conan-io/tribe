# Proposal: Do not automatically create a default profile if not existing.


| **Status**        | **Accepted**                                      |
|:------------------|:--------------------------------------------------|
| **RFC #**         | [024](https://github.com/conan-io/tribe/pull/24)  |
| **Submitted**     | 2021-06-02                                        |
| **Tribe votes**   | :thumbsup: (37) :thumbsdown: (0) :eyes: (3)       |


## Summary

When the "default" profile in the ``.conan/profiles`` folder, does not exist, and is required by some Conan command, raise
an error indicating that the profile doesn't exist, and telling the user to explicitly create it.

There will be one Conan command, like the current ``conan profile new default --detect`` (hopefully with
better UI) that will perform the autodetection. The result of the autodetection will not be considered stable.


## Motivation

In one of the initial Conan versions, the default ``libcxx=libstdc++`` was defined, also for ``gcc>=5`` compilers. At that time, this was the default standard library in many mainstream Linux distros **even
if you upgraded your compiler version to ``gcc>5``. It was a good default, provided the largest binary
compatibility, etc.

With the upgrade of the Linux distros and compiler versions, ``libcxx=libstdc++11`` became the default,
but it was not possible to change the Conan default profile without braking. This created a lot of pain,
because even if the automatically detected profile is displayed the first time is created, with bright
colors, users will most times just ignore it. Then, when trying to link Conan packages in their applications,
they get confusing link errors, which trace back to the libcxx incompatibility, difficult to understand.

This has been an important pain for years, that couldn't be changed without breaking many users.

Besides that, the approach has also shown some issues, as an auto-detection is by its nature, impossible to get it always right. Conan will assume some priorities, like in Windows the "default" compiler will be Visual Studio if installed, even if there is some gcc/mingw installed one, or prioritizing gcc over clang.
Also misdetections happen, and the latest compiler version you installed and want to use is not correctly detected, and then the default profile might be using the older one.

For production usage, using your own defined profiles has been the recommended approach for a long time.
In general, automating the detection of a default profile, while it sounds convenient for users, specially
beginners, excludes some relevant information that Conan users should be aware of.


## Detailed Design

After installing Conan, there will not be a default profile. The first Conan command that doesn't provide
a profile and needs such a default one, will raise an error indicating this. This same error will clearly
display the command necessary to automatically detect and create such a profile.

The command to create the profile will be more explicit about the detection and the results. It will do
nothing more than creating the default profile in the cache, as it does now with the ``conan profile new default --detect`` command.

No ``conan install ... --auto-detect-profile`` argument will be provided, as it defeats the whole purpose of this proposal.

The result of the automatic profile detection command will not be considered stable, and subject to change. This is necessary to avoid future ``libcxx=libstdc++`` problems, and other similar issues that will appear
as the compiler toolchains and the language itself keep evolving. For production, using your own defined
profiles is the recommended approach.


## Migration plan

Users using their own profiles, and not relying on the default one, nothing will change.
Users relying on the automatically detected default profile will need to change their commands. For developers, it is easy, as they can type the new command. For CI and other tools integration, the new UI
for the ``conan profile new default --detect`` can be provided in Conan 1.X to allow the smooth transition.
