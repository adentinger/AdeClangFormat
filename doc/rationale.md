# Rationale of AdeClangFormat

**TL;DR:** AdeClangFormat uses information provided by CMake to call `(git-)clang-format` and is designed to have a single config handling both automated setups like CI and irregular contributors who may not need/want to run `(git-)clang-format` checks locally.

--------------

Enforcing formatting rules with a CI build is desirable of course. For example, it improves readability, ensures all developers follow the same (hopefully sane!) formatting style, and avoids the common "the formatting shows that this piece of code comes from a different person/era" problem. It also has some very down-to-earth functions, such as checking for invalid (and potentially invisible!) characters inside the files. This is important particularly in open-source settings, where contributions come from many sources, many of which maintainers may not have control over. Irregular contributors are also likely to not want to wade through long style guides.

`clang-format` and `git-clang-format` allow running formatting checks and fixes on source files, however they are mostly just commandline tools, or can be triggered by an IDE (e.g. VSCode), which not all developers might use. This means enforcing formatting rules automatically needs to be controlled by the project's CI config. Since projects using CMake already describe which files they are composed of, this same CMake information can be leveraged for formatting checks for simple --and most importantly robust-- configurations.

AdeClangFormat is a simple CMake module which links that CMake information with the commandline tools, and is built with these use-cases in mind. Importantly, just like [CMAKE_<LANG>_CLANG_TIDY](https://cmake.org/cmake/help/latest/variable/CMAKE_LANG_CLANG_TIDY.html), it is designed so that the same CMake user configuration supports both CI setups, which require the checks enabled, and irregular contributor setups, which should not require `clang-tidy` to be installed and should not require the build to even be inside a Git repository.

## Inspiration

This CMake module was, in part, inspired by [this post by Ben Boeckel](https://discourse.cmake.org/t/cmake-pre-build-command/1083/11?u=anthonyd973), as well as [this CMake discourse thread](https://discourse.cmake.org/t/clang-format-integration/3358/6?u=anthonyd973).
