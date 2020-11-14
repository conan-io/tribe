# Proposal: Configurable and distributable toolchains

| **Status**        | **Proposed/Accepted/Deprecated** |
|:------------------|:---------------------------------------------|
| **RFC #**         | ####                                         |
| **Submitted**     | YYYY-MM-DD                                   |
| **Dependencies**  | RFC #, #                                     |

---

## Summary
Define a special type of conan package (profile-package) that can be used only in conan profiles or pose instead of conan profile, but have the ability to impose settings on the dependency graph and ability to update `settings.yml` file.


## Motivation
Conan profiles are a great way of sharing common settings and build requirements in your development team. However, the feature currently does not have enough flexibility, such as enforcing specific compile flags on all packages in the dependency graph. Also, profiles cannot be distributed as a conan package, so different channels must be use to distribute profile to your development team.
The motivation originated from [this Github issue comment](https://github.com/conan-io/conan/issues/7493#issuecomment-669135086). For convenience, the original comment is copied to the end of this file.


## Proposal
Define a special "profile-package" type of the conan package, as special conan package that:
- can impose settings on the dependency graph
- may have `build_requires` and `python_requires`, optionally even `requires`
    - for using common python utilities and for depending on already-existing conan packages, such as cmake_installer, emsdk_installer, android_ndk, ...
    - if we don't allow `requires` for such packages, then the build procedure will have to ensure that all tools are repackaged into the toolchain (which makes sense for repackaging emsdk_installer, android_ndk and similar, but I'm not so sure about the purpose of repackaging cmake - for such case it may make sense to allow `requires`, but in some kind of "restricted" form. We need to discuss this further).
- are optionally configurable
    - for example, a specific toolchain may have an option to enable/disable sanitizers, LTO, etc. The changed option would require different package IDs of the entire graph built using the toolchain-profile. Alternatively, we could require different toolchains for each change. This is how currently @vector-of-bool's [dds](https://vector-of-bool.github.io/docs/dds/guide/toolchains.html) does it, but I find this pretty annoying - I would prefer configurability.
- should be able to update `settings.yml`
    - this is a bit tricky, but it may be required for some toolchains to add specific architectures, OS options or compiler versions that do not exist in the default `settings.yml`
    - for example, let's consider an iOS toolchain that ensures that produced binaries are all fat. It's obviously wrong to advertise package architecture as `x86_64` as it will also contain `arm64` and other slices. However, it's possible to use the `ios_fat` architecture, which does not exist in the default `settings.yml`, so, an iOS toolchain should be able to register new architecture to `settings.yml` during its installation.
    - another example: Emscripten. Emscripten does not allow for link-compatibility between their minor versions, while the used clang compiler advertises the same version. For example, binaries built with emsdk 1.39.16 are not link-compatible with binaries built with emsdk 1.39.11, while both emsdk versions advertise clang 6.0.2 for their `fastcomp` backend and clang 11.0.0 for their `upstream` backend. This could be addressed by introducing special clang versions `6.0-emsdk-1.39.11`, `6.0-emsdk-1.39.16`, `11-emsdk-1.39.11`, `11-emsdk-1.39.16`, ..., in `settings.yml` on installation. Additionally, the emscripten toolchain may allow for choosing whether you want to enable emscripten threads and SIMD support, which require imposing specific compile flags on all packages in the dependency graph.
- can be distributed via Artifactory
    - a toolchain should be deployed to a conan repository on Artifactory and installed with `conan config install` or similar command. The installation would download the profile-package from the conan repository, create profiles associated with the package and trigger hooks that will allow for custom updates of `settings.yml` as described above.


## Alternative Approaches
Alternative approach, using only currently available conan feature, is completely described in [this Github comment](https://github.com/conan-io/conan/issues/7493#issuecomment-669135086). It's also added to the end of this file, for convenience.


## Detailed Design


## Open issues

- [conan issue #7493](https://github.com/conan-io/conan/issues/7493)
- [conan issue #3573](https://github.com/conan-io/conan/issues/3573)
- possibly [conan issue #4753](https://github.com/conan-io/conan/issues/4753), if observed from the point of view of build and host profiles. This would be replaced with build and host toolchains (profile-packages).


## Future Extensions

- the issue of minumum version of Visual Studio, discussed in [conan-tribe PR-5](https://github.com/conan-io/tribe/pull/5) would be completely avoided, as specific versions of MSVC could be handled with custom toolchains, as described in [this comment](https://github.com/conan-io/tribe/pull/5#issuecomment-727196995)
    
## Original comment that motivated the proposal, verbatim

> Also @DoDoENT we would be interested in your feedback for the toolchain() feature. This will be the proposed future integration with build systems in Conan 2.0.

I didn't play with this feature, but I've read the [documentation](https://docs.conan.io/en/latest/creating_packages/toolchains.html) as part of trying to find a workaround for this very issue and this got me confused. Basically the current `toolchain` feature of conan is the same thing as CMake build helper, with the difference that all cmake flags are encoded into a `conan_toolchain.cmake` file, instead of being given directly via command line. This looks very useful during the development of the package, but the name `toolchain` is misleading. Yes, it abuses the `CMAKE_TOOLCHAIN_FILE` of CMake to achieve its goal, but I would argue that this is not a "toolchain", but merely a "package-specific serialized build options".

Here is what I expect from the `toolchain` (as a word): _a set of tools, utilities, libraries and configuration of those needed to build **any** package_. This includes:
- compilers (`gcc`, `clang`, `msvc`, ...)
- tools (cmake, bazel, automake, ninja, ...)
- utilities (`ar` vs `llvm-ar`, `ranlib` vs `llvm-ranlib`, ...) 
- libraries
    - STL: libc++, libstdc++, libstdc++11, MS STL, ...
    - libc: glibc by versions, muslc by versions, bionic by versions
    - system libraries, for example: 
        - Windows SDK version: WinXP, Win7, Win10, ...
        - MacOS deployment target
        - iOS deployment target
        - Android minimum API level
        - Emscripten SDK version
        - ...
- configuration
    - compile flags (e.g. I want to have different toolchains for `centos-gcc-generic-x86-64` and `centos-gcc-haswell` - in the haswell case I want to enforce `-march=haswell` across entire dependency graph and make sure the package ID are different than in generic x64 case)
    - LTO (if I enable LTO, I want all packages in my dependency graph to be built with `-flto` or the equivalent `/ltcg` is msvc case - note that enabling/disabling such feature affects binary compatibility between packages depending on the compiler being used. For example, if LTO is not used, it's possible to link together packages built with slightly different compiler version (e.g. gcc 9.2 and gcc 9.3; msvc 19.23 and msvc 19.24, apple-clang 11.0.0 and apple-clang 11.0.3), but if LTO is used this is no longer possible (except for Linux clang, which has compatible bitcode between, e.g., 10.0.0 and 10.0.1))
    - PIC
    - sanitizers (address, leak, memory, undefined behaviour, ...)
    - some custom flags (for example I want to have different clang toolchains for `-Os` and `-Oz` optimization-for-size flags, in order to compare impacts of those flags on the entire codebase/dependency graph)
    - tool configuration (e.g. CMake variables)

Ideally, a proper toolchain should be a conan package itself, with its own options (for example, clang toolchain may have options to toggle LTO, AddressSanitizer, ControlFlowSanitizer, ...). The only difference to the "normal" should be the ability to impose settings to the rest of the dependency graph at the cost of having only a single toolchain per build context in the graph. At the moment, this is only possible via the command line or by using profiles.

As you can see, the toolchains are more related to profiles and build requirements than the concrete packages - this is what first confused me when I saw `toolchain` method as part of the package and why I argue that the name is misleading.

Actually, to be fair, most of the abilities I've described above are already possible with the current conan model, although it requires some orchestration between profiles and conan packages which requires some conan experience and can be very intimidating for the beginners. But it works quite well when configured properly. Here is how we currently do it at Microblink:

- there is a single conan package called `CMakeBuild` which contains common cmake utilities and compile flags we want to impose to all our packages while building them. Specific packages may override some flags, but this is clearly visible in the PR diff and requires the approval of the senior developer (this is rarely used). This package is basically a wrapper around [this open source project](https://github.com/microblink/build), but we also added some features that are specific for our in-house procedures.
- every package is required to have a direct dependency to `CMakeBuild` so that whenever package ID of `CMakeBuild` changes, the package gets rebuilt
- `CMakeBuild` is required to follow semantic versioning scheme, i.e. whenever a compile flag is changed that would alter link-compatibility or if we want to impose the new flag on the entire codebase, a major version needs to be raised. This is problematic for cases when you want to make a change to a single platform only. For example, I want to enable LTO for release builds for Windows: I add the flag into a cmake file, create a new `CMakeBuild` package with major version bumped and override the `CMakeBuild` package version at the most downstream level. This ensures that the entire dependency graph is rebuilt using LTO on windows, but it also causes rebuilds on other platforms (iOS, Android, Emscripten, ...) which had no changes. This wastes time and resources. In the ideal case, I would only want to make a new "windows" toolchain.
- versions of the compiler, min SDK levels, STL versions and OS versions are managed by the profile files that need to be kept in sync with `CMakeBuild` versions. This is the most tricky part, as it's very easy to have different information in `CMakeBuild` and in profile (e.g. we had an internal bug where our `CMakeBuild` ensured fat iOS binaries to be produced with iOS deployment target 8.0, while the ios profile stated that architecture is `x86_64` and deployment target 10.0).
- we ensure that everything is in sync by using a [custom cmake script that detects a profile from cmake settings](https://github.com/microblink/conan-build-helper/blob/master/mb_conan_build.cmake#L135), but this still requires that during development the developer correctly sets the cmake toolchain during cmake invocation. We have other helper scripts for that, but this is actually the part where the feature that conan currently calls a _toolchain_ could come handy.
- for cross-compilations, profiles additionally have a `build_requires` on conan package that actually contains the cross-toolchain (Emscripten or Android NDK). For iOS case, the "toolchain" is basically a repackaged `CMakeBuild`, but only those that bootstrap cmake into "iOS-mode", without those that may actually make binary different (this must be carefully repackaged because conan build_requirements do not affect package ID, and in this case, we want that).

So, I would seek-out to implement toolchains as an upgrade to profiles. Just like packages allow `conanfile.txt` for simple usages and `conanfile.py` for more complex use-cases (creating a package or more complex package consumer), I think that profiles should also be possible to exist in text-only simple form (just as is today) and in more complex "toolchain-package" form, as special conan packages that:
- can impose settings on the dependency graph
- may have `build_requires` and `python_requires`, optionally even `requires`
    - for using common python utilities and for depending on already-existing conan packages, such as cmake_installer, emsdk_installer, android_ndk, ...
    - if we don't allow `requires` for such packages, then the build procedure will have to ensure that all tools are repackaged into the toolchain (which makes sense for repackaging emsdk_installer, android_ndk and similar, but I'm not so sure about the purpose of repackaging cmake - for such case it may make sense to allow `requires`, but in some kind of "restricted" form. We need to discuss this further).
- are optionally configurable
    - for example, a specific toolchain may have an option to enable/disable sanitizers, LTO, etc. The changed option would require different package IDs of the entire graph built using the toolchain-profile. Alternatively, we could require different toolchains for each change. This is how currently @vector-of-bool's [dds](https://vector-of-bool.github.io/docs/dds/guide/toolchains.html) does it, but I find this pretty annoying - I would prefer configurability.
- should be able to update `settings.yml`
    - this is a bit tricky, but it may be required for some toolchains to add specific architectures, OS options or compiler versions that do not exist in the default `settings.yml`
    - for example, let's consider an iOS toolchain that ensures that produced binaries are all fat. It's obviously wrong to advertise package architecture as `x86_64` as it will also contain `arm64` and other slices. At Microblink, we use the `ios_fat` architecture, which does not exist in the default `settings.yml`, but we needed to manually add that to `settings.yml` and distribute it. Ideally, an iOS toolchain should be able to register new architecture to `settings.yml` during its installation.
    - another example: Emscripten. Emscripten does not allow for link-compatibility between their minor versions, while the used clang compiler advertises the same version. For example, binaries built with emsdk 1.39.16 are not link-compatible with binaries built with emsdk 1.39.11, while both emsdk versions advertise clang 6.0.2 for their `fastcomp` backend and clang 11.0.0 for their `upstream` backend. We currently address this issue by introducing special clang versions `6.0-emsdk-1.39.11`, `6.0-emsdk-1.39.16`, `11-emsdk-1.39.11`, `11-emsdk-1.39.16`, ..., in our `settings.yml`. Additionally, our emscripten internal toolchain allows for choosing whether you want to enable emscripten threads and SIMD support. We modelled that with OS options for the "Emscripten" OS in our `settings.yml`, but ideally a toolchain should be able to register these new possible compiler versions and OS settings entries to user's `settings.yml` upon toolchain installation.
- can be distributed via Artifactory
    - currently, we are distributing profiles and custom settings.yml via zip files built by our Jenkins and `CMakeBuild` and ios/android/emsdk "toolchain packages" via artifactory. It would be much easier if a toolchain could be deployed to a conan repository on Artifactory and installed as "config-package/toolchain/smart-profile/call-it-as-you-wish" with `conan config install`. The installation would do the same as it currently does, but it would also trigger hooks in "toolchain packages" that will allow for custom updates of `settings.yml` as described above.


