# AdeClangFormat

This CMake module is a simple, scalable and pragmatic helper for running either `git-clang-format` or `clang-format` in the context of a CMake build.

## Rationale of this repo

See [here](doc/rationale.md).

**TL;DR:** AdeClangFormat uses information provided by CMake to call `(git-)clang-format` and is designed to have a single config handling both automated setups like CI and irregular contributors who may not need/want to run `(git-)clang-format` checks locally.

## Usage

```cmake
# CMakeLists.txt

find_package(AdeClangFormat 1.1 CONFIG REQUIRED)

# Uses git-clang-format (preferred; see below)
ade_clang_format_git_targets(TARGETS
  target1
  target2)

# Uses clang-format (slightly simpler; does not require Git)
ade_clang_format_plain_targets(TARGETS
  target3
  target4)
```

These both introduce two main new targets (which you can run with `cmake --build <binary-dir> --target <target>`):
- `formatcheck`: Runs `(git-)clang-format` on the sources of all specified targets without actually modifying them. The target fails if `(git-)clang-format` finds formatting errors.
- `formatfix`: Runs `(git-)clang-format` on the sources of all specified targets, asking `(git-)clang-format` to fix them.

By default, building the target(s) specified to these CMake functions also runs the `(git-)clang-format` check to verify the formatting of the target's sources (but that can be turned off; see `NO_BUILD_DEPENDS` below). Useful to enforce formatting compliance, such as in a CI build.

Setup also requires specifying minimal information, usually in the shape of cache variables:

```bash
cmake \
  -DADE_CLANG_FORMAT_GIT:STRING='git;clang-format;--style=file:.clang-format' \
  -DADE_CLANG_FORMAT_GIT_REF_COMMIT:STRING='origin/develop' \
  -DADE_CLANG_FORMAT_PLAIN:STRING='clang-format;--style=file:.clang-format' \
  <other-flags>
```

## git-clang-format vs plain clang-format

See [here](doc/git-clang-format.md).

**TL;DR:** For scalability (e.g. accounting for `clang-format` version upgrades), always prefer `git-clang-format` (therefore `ade_clang_format_git_targets()`) to `clang-format` (`ade_clang_format_plain_targets()`) in CI/automated setups like AdeClangFormat is built for, even if `git-clang-format` requires slightly more setup.

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

...then either add the created `install/` directory into your CMake build's `CMAKE_PREFIX_PATH`, or copy the contents of the `install/` directory to a location where [`find_package()`](https://cmake.org/cmake/help/latest/command/find_package.html) will find them, such as a directory already within the `CMAKE_PREFIX_PATH`.

## Full usage

```cmake
ade_clang_format_git_targets(TARGETS [targets...]
  [BUILD_DEPENDS | NO_BUILD_DEPENDS]
  [COMMAND command [args...]]
  [CHECK_MAIN_TARGET <name>]
  [CHECK_POSTFIX <string>]
  [FIX_MAIN_TARGET <name>]
  [FIX_POSTFIX <string>]
  [REF_COMMIT <commit>]
)

ade_clang_format_plain_targets(TARGETS [targets...]
  [BUILD_DEPENDS | NO_BUILD_DEPENDS]
  [COMMAND command [args...]]
  [CHECK_MAIN_TARGET <name>]
  [CHECK_POSTFIX <string>]
  [FIX_MAIN_TARGET <name>]
  [FIX_POSTFIX <string>]
)
```

Sets up check and fix targets that invoke `(git-)clang-format` on the sources of the targets specified to `TARGETS`. Note that these functions may be invoked multiple times, as long as the same target is not specified again.

The environment variables that influence these calls are:

- `ADE_CLANG_FORMAT_GIT_REF_COMMIT`: Same as the cache variable of the same name. If specified and the cache variable of the same name is unspecified, this environment variable's value is used.

The cache variables that influence these calls are:

- `ADE_CLANG_FORMAT_GIT`/`ADE_CLANG_FORMAT_PLAIN`: Default value of the `COMMAND` option of `ade_clang_format_(git/plain)_targets()` if unspecified. This is a semicolon-delimited list of flags that run `(git-)clang-format`. Works the same way as [`CMAKE_<LANG>_CLANG_TIDY`](https://cmake.org/cmake/help/latest/variable/CMAKE_LANG_CLANG_TIDY.html) does. At the moment, this applies to sources of all languages on the specified targets, see [issue #6](https://github.com/adentinger/AdeClangFormat/issues/6). If unspecified and if the `COMMAND` option is also unspecified, then the invocation is a NO-OP (but the check and fix targets are not created to help detect configuration errors). This is similar to CMake's [`CMAKE_<LANG>_CLANG_TIDY`](https://cmake.org/cmake/help/latest/variable/CMAKE_LANG_CLANG_TIDY.html) and is useful to have a single invocation of `ade_clang_format_(git/plain)_targets()` that is able to handle both dev/CI setups, which need the formatting targets, _and_ irregular contributors who may not have `clang-format` installed or may not want to set it up.

- `ADE_CLANG_FORMAT_GIT_BUILD_DEPENDS`/`ADE_CLANG_FORMAT_PLAIN_BUILD_DEPENDS`: A boolean option which, when `ON`, causes invocations of `ade_clang_format_(git/plain)_targets()` to default to setting the `BUILD_DEPENDS` option, and, when `OFF`, causes invocations of `ade_clang_format_(git/plain)_targets()` to default to setting the `NO_BUILD_DEPENDS` option. When unspecified, this variable is considered to be `ON`.

- `ADE_CLANG_FORMAT_GIT_REF_COMMIT`: Affects `ade_clang_format_git_targets()` only. Default value of `REF_COMMIT`. If 1) this is unspecified, 2) the `REF_COMMIT` option is also unspecified, and 3) either `COMMAND` or `ADE_CLANG_FORMAT_GIT` are specified, then an error is raised (to help detect configuration errors).

The options are:

- `TARGETS`: The targets whose sources to setup `(git-)clang-format` for. The targets must already exist.

- `BUILD_DEPENDS`: Force `(git-)clang-format`'s check to be run on the target's sources when the target is built, regardless of the setting of the `ADE_CLANG_FORMAT_BUILD_DEPENDS` cache variable.

- `NO_BUILD_DEPENDS`: Force `(git-)clang-format`'s check _not_ to be run on the target's sources when the target is built, regardless of the setting of the `ADE_CLANG_FORMAT_BUILD_DEPENDS` cache variable.

- `COMMAND`: Force the command to run to be the specified command, regardless of the setting of the `ADE_CLANG_FORMAT_(GIT/PLAIN)` cache variable. If both this option and the `ADE_CLANG_FORMAT_(GIT/PLAIN)` cache variable are unspecified, no error is reported (but the check and fix targets are not created to help detect configuration errors). This is similar to CMake's [`CMAKE_<LANG>_CLANG_TIDY`](https://cmake.org/cmake/help/latest/variable/CMAKE_LANG_CLANG_TIDY.html) and is useful to have a single invocation of `ade_clang_format_(git/plain)_targets()` that is able to handle both dev/CI setups, which need the formatting targets, _and_ irregular contributors who may not have `(git-)clang-format` installed or may not want to set it up.

- `CHECK_MAIN_TARGET`: Specify the target that will run the `(git-)clang-format` check on the sources of the specified targets. If this is not specified, a default of `formatcheck` is used.

- `CHECK_POSTFIX`: At the moment, AdeClangFormat creates individual `(git-)clang-format` check targets for each of the specified targets. These targets' name is `<target-name>${CHECK_POSTFIX}`. If this is not specified, a default of `-formatcheck` is used. Change this if, for some extraordinary reason, there is a target naming conflict.

- `FIX_MAIN_TARGET`: Specify the target that will run the `(git-)clang-format` fix on the sources of the specified targets. If this is not specified, a default of `formatfix` is used.

- `FIX_POSTFIX`: At the moment, AdeClangFormat creates individual `(git-)clang-format` fix targets for each of the specified targets. These targets' name is `<target-name>${FIX_POSTFIX}`. If this is not specified, a default of `-formatfix` is used. Change this if, for some extraordinary reason, there is a target naming conflict.

- `REF_COMMIT`: For `ade_clang_format_git_targets()` only. Force the reference commit to be the value specified, regardless of the setting of the `ADE_CLANG_FORMAT_GIT_REF_COMMIT` cache variable. If 1) this is unspecified, 2) `ADE_CLANG_FORMAT_GIT_REF_COMMIT` is also unspecified, and 3) either `COMMAND` or `ADE_CLANG_FORMAT_GIT` are specified, then an error is raised (to help detect configuration errors).
