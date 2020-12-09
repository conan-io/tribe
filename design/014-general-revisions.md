
# Proposal: General - Make revisions enabled the default for Conan 2.0

| **Status**        | **Accepted**                                      |
|:------------------|:--------------------------------------------------|
| **RFC #**         | [014](https://github.com/conan-io/tribe/pull/14)  |
| **Submitted**     | 2020-11-25                                        |
| **Tribe votes**   | :thumbsup: (52) :thumbsdown: (0) :eyes: (2)       |

---

## Summary
Enable [package revisions](https://docs.conan.io/en/latest/versioning/revisions.html) as the default
and only behavior, and remove support for packages without revisions in Conan 2.0.

## Motivation
Package revisions have proved to work very well for achieving packages traceability and immutability.
Also, combined with [lockfiles](https://docs.conan.io/en/latest/versioning/lockfiles.html), this
feature is a powerful tool for achieving reproducible builds.

Maintaining two different modes in the current version is challenging for several reasons, like:
- Exactly the same command  may have different meaning and behavior depending on if revisions are
  enabled or not. That is the case of the `--update` argument for the `conan install` command. This
  fact can sometimes confuse users about how Conan should behave in certain situations.
- Each mode has its version of the server API, v1 for packages without revisions and v2 for revisions
  enabled. Besides the fact that the v2 API supports more features, making the server API v2 the
  default would make the maintenance simpler.
- Full decentralization of packages on the server side is impossible without revisions as a result of
  packages not being immutable, the same package reference can be completely different thing in
  different servers.

## Proposal
Use revisions enabled as the default mode in Conan 2.0 and remove support for packages without
revisions:
- All commands will assume that packages are using revisions, and behave accordingly.
- Produced files, like lockfiles, will contain the package revisions automatically.

## Detailed Design
Removing support for packages without revisions, means that Conan servers should use revisions as
well (Rest HTTP V2 support).