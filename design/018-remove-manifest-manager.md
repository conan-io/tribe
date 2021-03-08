# Proposal: Remove manifest check manager

| **Status**        | **Accepted**                                      |
|:------------------|:--------------------------------------------------|
| **RFC #**         | [018](https://github.com/conan-io/tribe/pull/18)  |
| **Submitted**     | 2021-02-04                                        |
| **Tribe votes**   | :thumbsup: (14) :thumbsdown: (0) :eyes: (38)      |

---


## Summary

Remove the manifest capture and checker functionality from the ``install`` and ``create`` commands.


## Motivation

Conan implements in ``install|create --manifests --manifests-interactive --verify`` arguments a functionality that captures the manifests of the packages into the
user project and is able to validate it they changed and raise accordingly.

This functionality seems to be quite outdated at the moment:
- It didn't get a single question, bug report or feature requests in years. Seems completely unused.
- It is not providing any security guarantee. Practically all Conan users concerned about security are building their own binaries and hosting them privately in their own server.
- New revision mechanism, computed as the hash of the manifest itself or the git commit, is in practice an implementation of this approach. Pinning exact revisions in your requirements, or using lockfiles guarantee that no new versions or artifacts are pulled in.


## Proposal

Remove this functionality from Conan as built-in feature.

## Detailed Design

This is still implementable as a hook. Not only possible, but actually it would be more realistic and useful:

- Hooks can be enabled in config, installed with ``conan config install`` and they will run automatically.
- No need for the users to provide ``--manifests`` or ``-verify`` on the command line, this is more fragile, users can forget.
- Hooks are way more configurable. Users could customize their hook to have a central storage of manifests, or could adapt it to capture licenses too, and similar things.
