
# Proposal: Tools - CMake 3.15

| **Status**        | **Accepted**                                      |
|:------------------|:--------------------------------------------------|
| **RFC #**         | [004](https://github.com/conan-io/tribe/pull/4)   |
| **Submitted**     | 2020-11-12                                        |
| **Tribe votes**   | 👍 (45) 👎 (7) 👀 (5)                              |

---

## Summary
Declare CMake 3.15 as the lowest supported CMake version.


## Motivation
CMake is one of the main external tool we call from Conan, transparent integration and
reliability should be a priority. It is important to declare a minimum supported
version so we can run all the tests and be confident that the user will get the
expected results and no regressions will be introduced.


## Proposal
CMake 3.15 was [released on July 2019](https://github.com/Kitware/CMake/releases/tag/v3.15.0). Significant additions are:

 * (CMake 3.14) _Visual Studio 16 2019_ generator.
 * Variable `CMAKE_PROJECT_INCLUDE` to add a file to be included after the `project()`
   call. This is used by Conan toolchains.
 * Variable `CMAKE_MSVC_RUNTIME_LIBRARY` to select the runtime library library used 
   when targeting MSVC ABI.
 
[Link to the full changelog](https://cmake.org/cmake/help/latest/release/3.15.html).


## Comments
This proposal is approved considering that the situation will change from the moment of approval to
when Conan 2.0 is released. At that moment, we will evaluate user's feedback to check if it's
necessary to add some workarounds or opt-out behavior for users chained to previous versions. 
