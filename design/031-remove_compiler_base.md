# Proposal: Remove "compiler.base" settings mechanism


| **Status**        |                                                   |
|:------------------|:--------------------------------------------------|
| **RFC #**         | [031](https://github.com/conan-io/tribe/pull/31)  |
| **Submitted**     | 2022-01-17                                        |
| **Tribe votes**   |                                                   |


## Summary

The mechanism of defining that a compiler such as “intel” in the settings.yml definition will kind of extend other existing compilers (such as “intel” having as a base either “gcc” in Linux or “msvc” in Windows) will be removed. Instead, a full definition of the compiler will be necessary, as the new IntelOneApi (via ``intel-cc`` setting has already been proposed in Conan 1.X).


## Motivation

When this feature was introduced, following feedback of users mainly using the Intel compiler, it seemed useful. Such users were reporting binary compatibility between packages built with Intel and packages built with the underlying gcc or Visual compiler, so, apparently, having a built-in definition of such base compatibility made sense.

But, since then, many of these users have now reported that such a feature is not adding value at all, and in fact in some cases it might be more confusing and problematic than the problem it tries to solve. Users are now directly defining specific settings per-package to explicitly define which configuration they want, and following their feedback, the new ``intel-cc`` compiler setting for IntelOneApi was defined, without such mechanism.

Removing the feature will simplify the implementation of generators and toolchains, and users wanting to implement some kind of fallback compatibility between different compilers will be able to use other mechanisms.


## Detailed Design

Note that this proposal doesn’t remove any ``compatible_packages`` mechanism. Actually, we are considering how to better represent such compatibility, with other better UX than coding it directly in the recipes ``package_id()`` method. This proposal only removes the ``settings.yml`` definition of the ``intel`` and ``mcst-lcc`` compilers, and replace them with a fully self-contained one, adding all the details they need to represent their binaries, without resorting to the definition of another compiler.


## Implementation details

The ``settings.yml`` file will be updated with the removal of the “base” subsettings, and the generators and code will remove all the checks of ``self.settings.compiler.base``.


## Migration plan

The Intel compiler already got a new ``intel-cc`` new compiler in 1.X that is fully independent. While it only supports at the moment IntelOneApi, it can be extended for previous versions if necessary.

The same process needs to be done for the ``mcst-lcc`` compiler in 1.X, defining a new independent one.

The resulting ``package_ids`` are different, so for the new settings there shouldn’t be any conflict with binaries.
