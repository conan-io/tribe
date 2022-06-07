# Proposal: Codebase - Python 3.6 required

| **Status**        | **Accepted**                                    |
|:------------------|:------------------------------------------------|
| **RFC #**         | [003](https://github.com/conan-io/tribe/pull/3) |
| **Submitted**     | 2020-11-20                                      |
| **Tribe votes**   | :+1: (59)                                       |

---

## Summary
Declare **Python 3.6** as the minimum Python version supported for Conan 2.0 launch.
Declare a Python version support policy for 12 months after the minimum supported version has officially become End Of Life (EOL).


## Motivation
Python ecosystem is moving forward,
[Python 3.5 has reached EOL](https://www.python.org/dev/peps/pep-0478/), and some
core Conan dependencies will stop to support it, like
[requests >2.25](https://requests.readthedocs.io/en/latest/community/updates/#id1).

Python 3.6 has also [some valuable features](https://docs.python.org/3/whatsnew/3.6.html)
that will help to improve the codebase and write more maintainable source:

 * Formatted string literals a.k.a. _f-strings_.
 * Type hints for classes and instance variables.

 There is also a support problem when Python officially declares versions EOL, because 
 dependencies stop supporting those versions and can break the Conan application. Conan
 can pin and restrict the usage of those dependencies newer versions, but that also has security implications,
 so it is not something that should be extended too much over time.


## Proposal
Python 3.6 will be the minimal supported version. Conan 2.0 is expected to be released
on 2021, by that date Python 3.6 will be the oldest release alive (EOL on December 2021) and
[Python 3.10 will be already released (October 2021)](https://www.python.org/dev/peps/pep-0619/).

About Linux distros: starting on Debian 10 Buster (July 2019), the [Python 3
version installed is Python 3.7](https://wiki.debian.org/Python). Ubuntu 18.04 (April 2018) already included Python 3.6.

Conan will drop support for Python versions 12 months **after** their official EOL date, that is,
it will continue testing and supporting one year beyond that date. When one version support
becomes deprecated, the next Python version will be used as the minimum tested and supported one. 


## Alternative Approaches
Newer Python versions provide several enhancements on type annotation that could help
to write and maintain code and some nice language features, but those are not _needed_
and can block Conan 2.0 from being deployed on some wide-spread systems.


## Detailed Design
As soon as we include some f-string in the sources, the test suite will start to fail
for any Python version older than 3.6.


## Open issues
Not known yet.


## Future Extensions
Not applicable.
