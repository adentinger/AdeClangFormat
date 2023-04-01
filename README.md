# AdeClangFormat

This CMake module is a simple and pragmatic helper for running `clang-format` in the context of a CMake build:

```cmake
find_package(AdeClangFormat 1.0 CONFIG REQUIRED)
ade_clang_format_targets(TARGETS
  target1
  target2)
```

This introduces two main new targets (which you can run with `cmake --build <binary-dir> --target <target>`):
- `formatcheck`: Runs `clang-format` on the sources of all specified targets without actually modifying them. The target fails if `clang-format` finds formatting errors.
- `formatfix`: Runs `clang-format` on the sources of all specified targets, asking `clang-format` to fix them.

The `clang-format` command to run can be specified via the CMake cache variable `ADE_CLANG_FORMAT` (similarly to CMake's [`CMAKE_<LANG>_CLANG_TIDY`](https://cmake.org/cmake/help/latest/variable/CMAKE_LANG_CLANG_TIDY.html) cache variable):

It is also possible to ask AdeClangFormat, while building the specified target(s), to run the corresponding `clang-format` check to verify the formatting of the target(s)'s sources. This is off by default, but is enabled by setting the `ADE_CLANG_FORMAT_BUILD_DEPENDS` option to `ON` (e.g. by setting it in the CMake cache on the command-line). Useful to enforce formatting compliance, such as in a CI build.

```bash
cmake \
    -DADE_CLANG_FORMAT:STRING='clang-format;--style=file:.clang-format' \
    -DADE_CLANG_FORMAT_BUILD_DEPENDS:BOOL=ON \
    <other-flags>
```

## Installation

Requires CMake >= 3.19. First run these commands...

```bash
git clone https://github.com/adentinger/AdeClangFormat && \
    cd AdeClangFormat && \
    mkdir build && \
    cmake -S . -B build -DCMAKE_INSTALL_PREFIX=install && \
    cmake --build build && \
    cd build && \
    ctest && \
    cd .. && \
    cmake --install build
```

...then either add the created `install/` directory into your CMake build's `CMAKE_PREFIX_PATH`, or copy the contents of the `install/` directory to a location where [`find_package()`](https://cmake.org/cmake/help/latest/command/find_package.html) will find them.

## Full usage

```cmake
ade_clang_format_targets(TARGETS [targets...]
	[BUILD_DEPENDS | NO_BUILD_DEPENDS]
	[COMMAND command [args...]]
	[CHECK_MAIN_TARGET <name>]
	[CHECK_POSTFIX <string>]
	[FIX_MAIN_TARGET <name>]
	[FIX_POSTFIX <string>]
)
```

Sets up check and fix targets that invoke `clang-format` on the sources of the targets specified to `TARGETS`. Note that this function may be invoked multiple times, as long as the same target is not specified again.

The cache variables that influence this call are:

- `ADE_CLANG_FORMAT`: Default value of the `COMMAND` option if unspecified. This is a semicolon-delimited list of flags that run `clang-format`. Works the same way as [`CMAKE_<LANG>_CLANG_TIDY`](https://cmake.org/cmake/help/latest/variable/CMAKE_LANG_CLANG_TIDY.html) does. At the moment, this applies to sources of all languages on the specified targets. If unspecified and if the `COMMAND` option is also unspecified on the invocation, the check and fix targets are created but are a no-op. This is similar to CMake's [`CMAKE_<LANG>_CLANG_TIDY`](https://cmake.org/cmake/help/latest/variable/CMAKE_LANG_CLANG_TIDY.html) and is useful to have a single invocation of `ade_clang_format_targets()` that is able to handle both dev/CI setups, which need the formatting targets, _and_ irregular contributors who may not have `clang-format` installed.

- `ADE_CLANG_FORMAT_BUILD_DEPENDS`: A boolean option which, when `ON`, causes invocations of `ade_clang_format_targets()` to default to setting the `BUILD_DEPENDS` option, and, when off, causes invocations of `ade_clang_format_targets()` to default to setting the `NO_BUILD_DEPENDS` option. When unspecified, this variable is considered to be `OFF`.

The options are:

- `TARGETS`: The targets whose sources to setup `clang-format` for. Must be existing targets.

- `BUILD_DEPENDS`: Force `clang-format`'s check to be run on the target's sources when the target is built, regardless of the setting of the `ADE_CLANG_FORMAT_BUILD_DEPENDS` cache variable.

- `NO_BUILD_DEPENDS`: Force `clang-format`'s check _not_ to be run on the target's sources when the target is built, regardless of the setting of the `ADE_CLANG_FORMAT_BUILD_DEPENDS` cache variable.

- `COMMAND`: Force the command to run to be the specified command, regardless of the setting of the `ADE_CLANG_FORMAT` cache variable. If both this option and the `ADE_CLANG_FORMAT` cache variable are unspecified, then the check and fix targets are created but are a no-op. This is similar to CMake's [`CMAKE_<LANG>_CLANG_TIDY`](https://cmake.org/cmake/help/latest/variable/CMAKE_LANG_CLANG_TIDY.html) and is useful to have a single invocation of `ade_clang_format_targets()` that is able to handle both dev/CI setups, which need the formatting targets, _and_ irregular contributors who may not have `clang-format` installed.

- `CHECK_MAIN_TARGET`: Specify the target that will run the `clang-format` check on the sources of the specified targets. If this is not specified, a default of `formatcheck` is used.

- `CHECK_POSTFIX`: At the moment, AdeClangFormat creates individual `clang-format` check targets for each of the specified targets. These targets' name is `<target-name>${CHECK_POSTFIX}`. If this is not specified, a default of `-formatcheck` is used.

- `FIX_MAIN_TARGET`: Specify the target that will run the `clang-format` fix on the sources of the specified targets. If this is not specified, a default of `formatfix` is used.

- `FIX_POSTFIX`: At the moment, AdeClangFormat creates individual `clang-format` fix targets for each of the specified targets. These targets' name is `<target-name>${FIX_POSTFIX}`. If this is not specified, a default of `-formatfix` is used.

## Inspiration

This CMake module was, in part, inspired by [this post by Ben Boeckel](https://discourse.cmake.org/t/cmake-pre-build-command/1083/11?u=anthonyd973), as well as [this CMake discourse thread](https://discourse.cmake.org/t/clang-format-integration/3358/6?u=anthonyd973).

