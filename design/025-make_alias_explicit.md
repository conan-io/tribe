# Proposal: Make alias requires always explicit.


| **Status**        | **Accepted**                                      |
|:------------------|:--------------------------------------------------|
| **RFC #**         | [025](https://github.com/conan-io/tribe/pull/25)  |
| **Submitted**     | 2021-06-02                                        |
| **Tribe votes**   | :thumbsup: (7) :thumbsdown: (0) :eyes: (31)       |



## Summary

Make "alias" requirements explicit, in the form ``requires = "pkg/(alias)@user/channel"``, where the
``alias`` can be any user string, as "latest".

Alias packages will remain named, implemented and managed the same. For a requires like ``requires = "pkg/(latest)@user/channel"``, there must exist a ``pkg/latest@user/channel`` package that will contain a single
``alias`` attribute pointing to the aliased real package. This doesn't change with respect the Conan 1.X status.


## Motivation

Alias is a complex and fragile feature, that has had many bug reports and issues in the last years. Most
of the source of this complexity and fragility is that there is no indication whatsoever in a ``require``
that such a requirement is not a real package, but kind of a symlink to the actual package that will be used.
So when a package declares something like ``requires = "pkg/latest@user/channel"``, it will not depend on anything called "latest", but on something like ``pkg/3.1.2@user/channel``.

This makes the dependency resolution challenging, because any requirement could be not an actual dependency,
but an alias to the real dependency. When there are complex graphs, with diamonds and possible conflicts,
then detecting the conflicts becomes challenging, because 2 different things like ``requires = "pkg/latest@user/channel"`` and ``requires = "pkg/1.0@user/channel"`` could be aliases to the same ``pkg/1.0.0@user/channel`` package. Not only the business logic of resolving those, but currently there is a good amount of code in Conan oriented to cache the alias resolution, as they are slow to resolve, and it is
common that  projects will use aliases in different packages, so this is necessary to keep good performance.

In the core of the issue, is the before mentioned lack of information in the requirement itself. Contrary
to the version ranges requirements, which define ``requires = "pkg/[>1.0]@user/channel"`` and the brackets
indicate that such dependency is not a real package, but it could resolve to something like 1.1, there is nothing in "alias" requirements that provide such a hint.


## Proposal

Introduce an alternative notation, like ``requires = "pkg/(alias)@user/channel"`` to indicate that such
requirement is not a real package, but an alias that will be resolved to something different, like ``pkg/1.0.2@user/channel``. This will establish a parallelism between version ranges and alias, make explicit to package writers and most importantly, readers, the intention to depend on a moving target that will be defined by an alias. It will also allow to simplify the dependency resolution algorithm, making it less fragile and probably faster.


## Detailed Design

While the ``requires`` syntax will allow ``()`` characters, the package names and versions will remain with the same restrictions, real package versions will not be allowed to contain the ``()`` characters, in the same way they do not allow the ``[]`` brackets.

Current resolution features will be maintained:

- It is possible to have transitive alias (one alias package aliasing other ones). The only requirement is that the alias definition should also be explicit: ``alias = "pkg/(otheralias)@user/channel"``
- Alias never appear in the final dependency graph, they are discarded in the graph evaluation process.
- Alias do not have settings, options, or any configuration. They cannot have python_requires, they are a recipe with 1 single class attribute ``alias``.


## Migration plan

The introduction of the ``()`` syntax might be introduced in Conan 1.X requirements, with no effect rather than filtering them out and ignoring them (they will still be resolved correctly, up to the limitations of Conan 1.X alias dependency resolution), keeping Conan 1.X working and allow the smooth transition to Conan 2.0.
