# git-clang-format vs plain clang-format

**TL;DR:** For scalability (e.g. accounting for `clang-format` version upgrades), always prefer `git-clang-format` to `clang-format` in CI/automated setups like AdeClangFormat is built for, even if `git-clang-format` requires slightly more setup.

------------

When using `clang-format` directly with a CMake module like AdeClangFormat, it is generally difficult to restrict which files to run `clang-format` on. This is because CMake generally cares about _targets_ rather than individual _files_. This means introducing `clang-format` to an existing codebase is likely to require utter annhilation of version control history, because in such a codebase every other line would likely have a formatting error. Also, introducing plain `clang-format` is required to be done in a single, _huge_ step, rather than being a gradual adoption.

What's more, [this answer by famous CMake maintainer Ben Boeckel](https://discourse.cmake.org/t/clang-format-integration/3358/6?u=anthonyd973) highlights the fact that `clang-format` is likely to want to format files differently accross versions, meaning updating `clang-format` will cause many formatting errors to appear in the codebase and require fixing all over again.

Instead, [`git-clang-format`](https://clang.llvm.org/docs/ClangFormat.html#git-integration), which is usually installed alongside `clang-format`, runs plain `clang-format` but ignores errors on lines that were not modified since a certain commit specified by the user (e.g. `main`, `develop`, or `HEAD`). This means only _new_ code is required to be properly formatted, solving the preceding issues.

However, naturally, running this requires the user to specify what this reference commit is. It also requires the CMake project to be inside a Git repo (which would not work inside downloaded source tarballs), and requires `git` to be available in the `PATH` so that `git-clang-format` may find it.
