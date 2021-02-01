# Proposal: References will be lowercase

| **Status**        | **Accepted**                                      |
|:------------------|:--------------------------------------------------|
| **RFC #**         | [017](https://github.com/conan-io/tribe/pull/17)  |
| **Submitted**     | 2020-12-18                                        |
| **Tribe votes**   | :thumbsup: (47) :thumbsdown: (3) :eyes: (0)       |

---


## Summary

Recipes references (package name, version, user and channel) should be lowercase. They will continue accepting other characters, digits(0-9), underscore, dots, etc. but letters (a-zA-Z) should be always lowercase. Trying to create a package with uppercase will raise an error (A temporary opt-out will be enabled to allow teams to migrate their packages gradually)


## Motivation

Conan 1.X is case sensitive, Boost, boost, BOOST are all different package names corresponding to completely different things. They cannot be overridden, they will not conflict with each other (in the graph sense, they will certainly produce errors at link time if they provide the same library names and symbols).

The server side will store them as different items, they will be retrieved as different packages with “conan search” command, and they can be installed independently with “conan install”.

This behavior generates a lot of issues, bugs and problems:

### Lack of conflicts

If package names are case sensitive, then it is possible that one package depends on a given boost/1.72 package and another one depends on BOOST/1.72 package. If they are different packages, the graph will be correct, and the errors will appear at build time, when library and symbol conflicts will happen.

As they are different, it is not possible to do overrides, and trying to force a common package from the downstream consumers will fail.

If the casing is not enforced, it is a matter of time that different communities generate packages for the same libraries (Boost, Poco, OpenCV, etc), that just differs in the casing, making them unusable (or at least very hard to swap), for users that were requiring packages with other casing.

### OS specific issues

Some OS, like Windows, have case-insensitive filesystems, making it impossible in practice to store different packages with different casing in the Conan cache.

### Security issues

Typo-squatting is a very well known attack vector for package managers. Having a package that just slightly differs from the “official” one and resolves differently is a way to deploy in developer machines different kinds of malware. Uppercase/lowercase differences are the simplest and most common differences, because they are not even typos, but a completely valid spelling. Let's say that ConanCenter contains a package called “boost/1.72”, malicious users could upload to other remotes also used by the community (but not as reviewed as ConanCenter) a “Boost/1.72” package just waiting there for users that spell it that way in their recipes.


## Proposal

All package references, (including the name, version, user and channel) should be unique, unambiguous and not confusing for users, reducing the cognitive effort. While making the package names case insensitive might alleviate issue number 1 (“Lack of conflicts”), the security issues still persist, and the cognitive overhead for users will still be there.

Then, Conan 2.0 will enforce recipes references (package name, version, user and channel) to be lowercase. They will continue accepting other characters, digits, underscore, dots, etc. but letters inside references should be always lowercase. Any attempt to create a package that is not lowercase will fail. If other mechanisms are employed (directly editing the Conan cache, or changing the server names) to change the reference of any package, that will be a broken one with undefined behavior.

## Detailed Design

One of the challenges of using packages with different casing for same name is the conflicts they generate while installing in the Conan cache in some operating systems. Conan 2.0 will provide a new cache layout that will be able to store these packages with different casing without conflicts. They will still be different, independent packages, and it will not be possible to use them in the same graph, as they will not conflict in the graph, but produce duplicated symbols while linking, but it will be possible to be installed in the cache, highly facilitating the migration to lowercase packages.

## Migration notes

As changing package names might take some time, Conan 2.0 will introduce a temporary opt-in in conan.conf to allow creating packages non-lowercase. This temporary opt-in doesn’t imply any behavior, Conan will still be case-sensitive, so they will be treated as completely different packages. But as commented above, the new cache will allow to install them without conflict. This change will make the migration to lowercase package names exactly the same as a major version bump, which is a well known process that can be done with reasonably planning. This opt-in will be removed in later Conan 2.X version, at least 6 months after the Conan 2.0 release (which will still require at least 6 months).
