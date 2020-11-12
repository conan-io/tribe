
# Proposal: Tools - Visual Studio 2017 (15.2) required

| **Status**        | **Proposed/Accepted/Deprecated** |
|:------------------|:---------------------------------------------|
| **RFC #**         | ####                                         |
| **Submitted**     | YYYY-MM-DD                                   |
| **Dependencies**  | RFC #, #                                     |

---

## Summary
Declare **Microsoft Visual Studio 2017 (15.2)** as the minimal version supported.


## Motivation
Visual Studio 2017 (15.2) was [released on May 2017](https://docs.microsoft.com/visualstudio/releasenotes/vs2017-relnotes-v15.2). This version allows to install
side-by-side different versions of the MSVC toolset, it allows you to [build binaries
that are fully compatible with the ones built using older versions](https://devblogs.microsoft.com/cppblog/stuck-on-an-older-toolset-version-move-to-visual-studio-2015-without-upgrading-your-toolset/).

Visual Studio 2017 (15.2) includes [`wswhere` application](https://github.com/microsoft/vswhere). 
This tool is needed Conan to locate Visual Studio installation and toolsets available (link to proposal required).


## Proposal
Declare **Visual Studio 2017 (15.2)** as the minimal version supported.


## Alternative Approaches
Conan migth need Visual Studio 2017 (15.2) **installed**, but it can target older 
versions (`vswhere` is able to find older installations too).


## Detailed Design


## Open issues
 * Evidence of CLI changes (calling MSBuild or other tools), format changes 
   in the `.props` files or the data available to them (used in Conan generators
   and toolchains). Besides locating the installation, is there any reason not to
   support older versions?

 * MSBuild doesn't support long paths until version 16.0 ([link](https://github.com/dotnet/msbuild/issues/53#issuecomment-459062618)).


## Future Extensions
