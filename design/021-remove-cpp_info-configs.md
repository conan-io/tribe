# Proposal: Remove cpp_info.config multi-configuration definition


| **Status**        |                                                   |
|:------------------|:--------------------------------------------------|
| **RFC #**         | [021](https://github.com/conan-io/tribe/pull/21)  |
| **Submitted**     | 2021-03-15                                        |
| **Tribe votes**   |                                                   |



## Summary

Remove the ``cpp_info`` multi-configuration definition that can be used in ``package_info()`` as:

self.cpp_info.release.libs = ["my_library"]
self.cpp_info.debug.libs = ["my_library_d"]


## Motivation

This feature was designed in Conan early stages, to allow the creation of multi-configuration packages, that is, packages that can contain both Debug and Release binaries. These packages will be independent of the ``build_type`` setting, either not declaring it or removing it from the ``package_id()``. They build all the configurations, one after the other in the ``build()`` method:

```python
def build(self):
    cmake = CMake(self)
    if cmake.is_multi_configuration:
        cmd = 'cmake "%s" %s' % (self.source_folder, cmake.command_line)
        self.run(cmd)
        self.run("cmake --build . --config Debug")
        self.run("cmake --build . --config Release")
        self.run("cmake --build . --config RelWithDebInfo")
        self.run("cmake --build . --config MinSizeRel")

def package_id(self):
    del self.info.settings.build_type
```


This approach has several disadvantages:

- It makes packages much heavier and inefficient. It doesn’t matter if only a Release version of a consumer is going to be built, it will need to fetch and unzip a much larger package, wasting a lot more of resources in CI or developers machines.
- It makes packages less secure, as it is much more complicated to release only Release artifacts, and unexpectedly leaking Debug artifacts with debug information embedded might happen easier.
- It makes packages more error prone to build, and more expensive to rebuild. If something goes wrong with any of the multiple configurations, it will be necessary to rebuild all of them. If something goes wrong in the uploads, downloads, etc., it will be necessary to rebuild all of them.
- It makes the Conan codebase more bloated, and requires more maintenance, slowing down development in this area of the codebase.

There is some evidence that this feature might not be used. The Conan ecosystem has largely adopted the one configuration per binary package paradigm. ConanCenter doesn’t use this feature at all. There have been no issues about this feature in years. Some generators like ``xcode`` or ``premake`` don’t even implement this feature, so even if packages declare it, its information will be lost by consumers.


## Proposal

Remove the cpp_info multi-configuration support from Conan 2.0


## Migration plan

Users relying on this feature will need to split their packages in single-configuration binary packages, using the ``build_type`` setting.
