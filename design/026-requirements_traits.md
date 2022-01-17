# Proposal: Define requirements new traits to qualify and enhance the Conan dependencies model


| **Status**        | **Accepted**                                      |
|:------------------|:--------------------------------------------------|
| **RFC #**         | [026](https://github.com/conan-io/tribe/pull/26)  |
| **Submitted**     | 2021-07-06                                        |
| **Tribe votes**   | :thumbsup: (33) :thumbsdown: (0) :eyes: (1)       |



## Summary

Introduce the concept of "requirements traits", to better model the relations between packages. Requirement traits are specifiers of the Conan ``requires`` that can be defined in recipes. These traits can allow defining the current Conan 1.X ``build_requires`` behavior with some ``requires`` traits.

This proposal does not affect ``python_requires``, because of their nature and evaluation at recipe parsing time, they are very different to regular ``requires`` and ``build_requires``, and they would not benefit from this proposal.

Requirement traits will define things like headers inclusions, library linkage, runtime dependency, if the headers visibility should propagate downstream, if the require should propagate downstream (and can conflict) or not, if it is a build tool or a test library, etc.

Package recipes will keep defining their direct dependencies in the same way they were doing in Conan 1.X, but with the possibility of specifying more traits (might not be necessary in many cases, the defaults should be fine). But recipes will have access, once the graph is evaluated, to the ``requires`` of all their dependencies, both direct(declared) and indirect ones (transitive), describing the relation they have with those dependencies with these traits.


## Motivation

Conan's main declaration of dependencies is the ``requires`` relationship. Mostly 2 variants of it are also available in Conan 1.X: ``build_requires`` that aim to express a dependency to a build tool, like cmake, and ``private`` which is discouraged for most cases.

But this model does not provide enough information to describe some specifics of the C/C++ compilation model, for example in the general case, when one library is linking another library, the consumers of the former shouldn’t have visibility over the headers of the later one. A ``requires`` trait called ``transitive_headers`` could control this visibility.
It seems the potential combinations of dependencies usage is too high to consider approaches like the current one, defining high level names as ``build_requires``, ``test_requires``, ``requires_shared``, etc. does not scale.


## Detailed Design

The proposed traits are arguments to the ``requires``, like:

```python
def requirements(self):
    self.requires(ref, headers=True, libs=True, run=None, visible=True, …)
```

There are defaults defined, so the same previous Conan 1.X syntax of class attribute ``requires ="ref"`` and ``self.requires(ref)`` is still possible.

Reminder, following the GNU standard naming:

- Build context: The machine in which the current Conan process and build is running. If we are cross-compiling in a Windows box for an app that will run in a Linux RaspberryPI environment, the build context is the Windows machine.
- Host context: The machine that will run the apps and libraries being built, in this example the host context is the Linux running in the RaspberryPI


The traits are:

- ``headers`` (default True): This trait indicates that there are headers that are going to be #included from this package at compile time. The dependency will be in the host context.
- ``libs`` (default True): The dependency contains some library or artifact that will be used at link time of the consumer. The dependency will be in the host context. This trait will be true for direct shared and static libraries, but could be false for indirect static libraries that are consumed via a shared library.
- ``build`` (default False): This dependency is a build tool, an application or executable, like cmake, that is used exclusively at build time, it is not linked/embedded into binaries, and will be in the build context.
- ``run`` (default None): This dependency contains some executables, either apps or shared libraries that need to be available to execute (typically in the path, or other system env-vars). By default is None, indicating that Conan can try to deduce if the dependency is needed to execute (if ``options.shared`` is True). This trait can be True for ``build=False``, in that case, the package will contain some executables that can run in the host system when installing it, typically like an end-user application. This trait can be True for ``build=True``, the package will contain executables that will run in the build context, typically while being used to build other packages.
- ``visible`` (default True): This ``require`` will be propagated downstream, even if it doesn’t propagate ``headers``, ``libs`` or ``run`` traits. Requirements that propagate downstream can cause version conflicts. This is by default True, because in most cases, having 2 different versions of the same library in the same dependency graph is at least complicated, if not directly violating ODR or causing linking errors.
- ``transitive_headers`` (default None): The headers of the dependency will be visible downstream or not. The default None allows Conan to auto detect this trait, for example, if the current package is a ``header-only`` one, and it depends on another library (header only, or static/shared), the headers of the transitive dependency must be available and used in the ``-I<includedirs>`` compilation downstream.
- ``transitive_libs`` (default None): The libraries to link with of the dependency will be visible downstream or not. The default None allows Conan to auto detect this trait, for example, if the current package is a ``header-only`` one, and it depends on another library (header only, or static/shared), the libraries of the transitive dependency must be available and used in the ``-I<libs>`` and ``-L<libdirs>`` compilation downstream.
- ``test`` (default False): this requirement is a test library or framework, like Catch2 or gtest. It is mostly a library that needs to be included and linked, but that will not be propagated downstream.
- ``package_id`` (default None): if the recipe wants to specify how the dependency version affects the current package ``package_id``, can be directly specified here. While it could be also done in the ``package_id()`` method, it seems simpler to be able to specify it in the ``requires`` while avoiding some ambiguities.
- ``force`` (default False): This ``requires`` will force its version in the dependency graph upstream, overriding other existing versions even of transitive dependencies, and also solving potential existing conflicts.
- ``override`` (default False): The same as the ``force`` trait, but not adding a ``direct`` dependency. If there is no transitive dependency to override, this ``require`` will be discarded. This trait only exists at the time of defining a ``requires``, but it will not exist as an actual ``requires`` once the graph is fully evaluated
- ``direct`` (default True): If the dependency is a direct one, that is, it has explicitly been declared by the current recipe, or if it is a transitive one.

Note: In most cases, for normal library requirements, it won’t be necessary to specify any special trait, just using the default ``self.requires(ref)`` or ``requires = "ref"`` will be enough, as the default traits, working together with the package type (shared/static/header-only) can automatically deduce and propagate correctly the information.

The requires and actual dependencies those requires are pointing to are accessible from recipes and tools like:

```python
def generate(self):
      for require, dependency in self.dependencies.items():
           require.direct # boolean
           require.headers # boolean
           require.libs # boolean
           require.build # boolean
           require.run # boolean
```

``build_requires`` will remain to be a high-level definition of requirements, but internally it is implemented as:

```python
req = Requirement(ref, headers=False, libs=False, build=True, run=True, visible=False,
                  package_id_mode=None, transitive_headers=None, test=False,
                  force=False, override=False, direct=True)
```


``build_requires`` are intended to be used for tools like ``cmake``. The important traits are:

- ``headers=False``: A ``build_require`` doesn’t provide headers, it is just an executable tool, like cmake
- ``libs=False``: A ``build_require`` doesn’t provide libraries to link, it is just an executable tool, like cmake.
- ``build=True``: A ``build_require`` runs in the build context.
- ``run=True``: A ``build_require`` typically contains something to run, like an executable.
- ``visible=False``: A ``build_require`` is not propagated to the consumers.


For the existing Conan 1.X ``self.build_requires(<ref>, force_host_context=True)``, the ``test_requires`` high level definition is proposed, intended for testing libraries like Catch2 or gtest. It will be internally equivalent to:

```python
req = Requirement(ref, headers=True, libs=True, build=False, run=None, visible=False,
                  package_id_mode=None, transitive_headers=None, test=True,
                  force=False, override=False, direct=True)
```

The important traits of ``test_requires`` are:

- ``headers=True``: A ``test_requires`` like Catch2 will provide headers to include
- ``libs=True``: A ``test_requires`` like provide libraries. Basically a ``test_requires`` is often a library-like package.
- ``build=False``: A ``test_requires`` is in the host context, not the build one, and it needs to libs with the host binaries
- ``run=None``: A ``test_requires`` can be a shared or static, and None allows this trait to be auto-deduced based on package type.
- ``visible=False``: A ``test_requires`` is not visible to the consumers.


This proposal tries to focus on the requirements traits definitions, but not yet on how these traits propagate down the graph, that will require a more elaborated proposal. Let's try to think first in terms of the relation between 2 packages, define the semantics of the traits.


## Implementation details

From the implementation point of view, there is not much detail yet. The interface will be the same one that general ``requires`` have now, just adding new arguments to the ``self.requires(.... visible=False, direct=True, ….)``.

A big part of the interface has already been exposed in Conan 1.38 with the ``self.dependencies`` attribute, this proposal will increase the number of traits that ``requires`` have.


## Migration plan

In order to smooth the upgrade to Conan 2.0, a ``**kwargs`` unused argument might be added to ``self.requires()`` in Conan 1.x, even if whatever is passed there is completely ignored in Conan 1.X, but that would allow to start specifying required traits in Conan 1.X recipes.

The explicit, high-level ``test_requires`` might also be introduced in Conan 1.X as an alias to ``self.build_requires(...., force_host_context=True)``.
