# Proposal: ABI Groups for packages

| **Status**        | **Proposed/Accepted/Deprecated** |
|:------------------|:---------------------------------------------|
| **RFC #**         | ####                                         |
| **Submitted**     | YYYY-MM-DD                                   |
| **Dependencies**  | RFC #, #                                     |

---

## Summary
Add an option to allow conan packages built with different - but ABI-compatible -
compilers to be used together.


## Motivation
Currently conan packages built with different compilers cannot be used in projects
using a different compiler, even if the generated binary is ABI-compatible.
While building all dependencies of a project with the same compiler can have some
clear benefits, not user scenarios warrant such constraint.
In fact, this fine-grained compiler requirement easily leads to bloat if the same
packages are used from projects using different compiler versions.
Additionally, we will lose out on pre-built binaries if the project switches to a compiler
version that is newer than the ones available in the conan-center-index CI.

Conan packages currently can work around this problem by overriding the `package_id`
function, but that is at the package creator's discretion and beyond control of the
package consumers.

## Proposal
* Introduce ABI versions for the various compiler versions/stdlib combinations.
* Add an additonal mechanism to opt-in/out on exact package compiler matches on a
per-project basis, with a default entry on the conan profiles.



## Alternative Approaches
Keep relying on `package_id` at the package creators' discretion.

## Detailed Design
TODO


## Open issues


## Future Extensions
