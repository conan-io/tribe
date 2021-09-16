# Proposal: Middleware to customize all recipes

| **Status**        | **Proposed** |
|:------------------|:---------------------------------------------|
| **RFC #**         | ####                                         |
| **Submitted**     | 2021-09-15                                   |

---

## Summary
Allow users to inject a layer or layers of code between Conan and all recipes.
For example, this would allow users to customize the build() and package() methods,
even for the recipes in CCI.  This provides support for advanced features
like multiple architectures without requiring any commitment to a particular approach.


## Motivation
Conan currently does not support multiple architecture (multiarch) binaries.
iOS and macOS call these universal binaries (and the tool is named *lipo*).
Android seems to support some sort of multiarch, although I'm unfamiliar with it.

This has been discussed, although these issues are unresolved:

<https://github.com/conan-io/conan/issues/1047>
<https://github.com/conan-io/conan/issues/6384>

With the addition of ARM to macOS last year, I consider this an urgent issue
and I imagine others would like to see some progress on this topic.

## Proposal
A multiarch proof of concept (as a hook, see below) ended up touching a fair
amount of Conan's internals, although it was fairly successful at compiling
universal CCI.  This inspired a general purpose extension to Conan where
multiarch support could be written like a recipe, with wrapper build() and package()
methods.  The support comes in a new class called ConanMiddleware.

By keeping all tool specific code outside Conan, this proposal allows
user customization without maintenance or compatibility concerns of trying to
implement multiarch directly.

A simpler use case of middleware is code signing.  The middleware could inject
the certificate name (from settings) in package_id() and in build() perform
code signing of all shared libraries and executables after the recipe's build().



## Alternative Approaches
Other users seem to run conan for each architecture and run lipo from a script.

In looking for a more integrated solution (with binary packages containing
universal binaries) I tried using a hook.  This was ugly, to say the least,
and highly dependent on Conan's internals.
<https://github.com/gmeeker/conan-multi-build-hook>

For macOS/iOS you can use CMake and the Xcode generator with a toolchain to
set CMAKE_OSX_ARCHITECTURES.  However, not all packages use CMake, some upstream
projects use CMake but are not compatible with the Xcode generator (libpng I believe)
or perform process checks that are not compatible with multiarch (libwebp).

This approach here (building each arch separately and running lipo) could be
performed by each recipe, without using the middleware approach.


## Detailed Design

A proof of concept implementation is in the *middleware* branch here:
<https://github.com/gmeeker/conan>

This only a proof of concept to demonstate that CCI recipes can be built as
multiarch.  This code is not intended to be production ready, but to show
that this proposal is even possible.

macOS/iOS multiarch middleware is here:
<https://github.com/gmeeker/conan-lipo-middleware>

Universal profile:

```
[settings]
os=Macos
os_build=Macos
os.version=10.13
multiarch=x86_64 armv8
arch=x86_64
arch_build=x86_64
compiler=apple-clang
compiler.version=12.0
compiler.libcxx=libc++
build_type=Release
compiler.cppstd=11
[options]
[build_requires]
lipo-multiarch-middleware/0.1@
[env]
```

lipo-multiarch-middleware connects the multiarch setting to lipo-middleware.

```
from conans import ConanFile

class Pkg(ConanFile):
    name = "lipo-multiarch-middleware"
    version = "0.1"
    python_requires = "lipo-middleware/0.1"
    settings = "multiarch"
    build_policy = "missing"

    def middleware(self):
        Lipo = self.python_requires["lipo-middleware"].module.Lipo
        multiarch = self.settings.multiarch
        if multiarch:
            def factory(conanfile):
                return Lipo(conanfile, variants=multiarch)
            return factory
        return None
```

Alternatively, the code could provide more detailed settings, such as
per-arch SDK or deployment targets.  (Providing multiple profiles would be
a future extension to avoid providing configuration as code.)

Here, a build subfolder is referred to as a variant rather than arch, because other
settings can be architecture dependent (e.g. macOS 10.13 for x86_64, 11.0 for ARM).
Multiarch is supported through a ConanVariantsMiddleware
subclass which manages subfolders for each variant, and duplicates a ConanFile
instance with modified settings.  These are different than build folders
because we are still building against dependencies with universal binaries,
not building the x86 and ARM dependency chains separately.

Some changes are required to some tools that detect if a settings argument
is a conanfile.  Although the middleware wraps the conanfile, it's not using
ABC (which would make conan think multiple ConanFile's are in a file) and
isinstance() will fail.  These tools now use duck typing.


## Open issues
Loading the middleware seems a little clunky, and I'm not sure how things
will look for Conan provided middleware (for example, if Conan eventually
provide a Lipo middleware).  Possibly the profile need ``[middleware]``
syntax, similar to ``[build_requires]``.


## Future Extensions
Instead of introducing a new multiarch setting, we could allow arch lists:
```
[settings]
arch=[x86, x86_64, armv7, armv8]
```

This proposal requires that the user creates a new package for detailed
variant settings.  A more flexible approach could be to pass multiple
profiles on the command line.
