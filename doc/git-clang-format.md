# git-clang-format vs plain clang-format

**TL;DR:** For scalability (e.g. accounting for `clang-format` version upgrades), always prefer `git-clang-format` (therefore `ade_clang_format_git_targets()`) to `clang-format` (`ade_clang_format_plain_targets()`) in CI/automated setups like AdeClangFormat is built for, even if `git-clang-format` requires slightly more setup.

------------

When using `clang-format` directly with a CMake module like AdeClangFormat, it is generally difficult to restrict which files to run `clang-format` on. This is because CMake generally cares about _targets_ rather than individual _files_. This means introducing `clang-format` to an existing codebase is likely to require utter annhilation of version control history, because in such a codebase every other line would likely have a formatting error. Also, introducing plain `clang-format` is required to be done in a single, _huge_ step, rather than being a gradual adoption.

What's more, [this answer by famous CMake maintainer Ben Boeckel](https://discourse.cmake.org/t/clang-format-integration/3358/6?u=anthonyd973) highlights the fact that `clang-format` is likely to want to format files differently accross versions, meaning updating `clang-format` will likely cause many formatting errors to appear in the codebase and require fixing all over again.

Instead, [`git-clang-format`](https://clang.llvm.org/docs/ClangFormat.html#git-integration), which is usually installed alongside `clang-format`, runs plain `clang-format` but ignores errors on lines that were not modified since a certain commit specified by the user (e.g. `main`, `origin/develop`, `HEAD~3`, ...). This means only _new_ code is required to be properly formatted, mostly solving the preceding issues [^1].

However, naturally, running this requires the user to specify what this reference commit is (e.g. "the commit to merge to"). It also requires the CMake project to be inside a Git repo (which would not work inside downloaded source tarballs), and requires `git-clang-format` to be available in the `PATH` so that `git clang-format` works.

So prefer using `ade_clang_format_git_targets()` to `ade_clang_format_plain_targets()`.

[^1]: There are still a couple of _much_ smaller issues with this. If a file is rejected by the CI, it is very possible that the contributor would either 1) manually fix formatting errors by copying and applying `git-clang-format`'s output found in the failed CI job's output, which may deter them from finishing their contribution, or 2) run `clang-format` on the entire file, which would introduce some version control noise. Instead, if you are interested in solving that exact problem, see issue #25.
