# Stilt toolchains

The clang that `stilt build --release` compiles C with. Each version of `stilt` names one
release of clang, and the URL and SHA-256 of an archive of it for each host. The first release
build on a machine downloads that archive, checks its hash, and unpacks it into
`~/.stilt/toolchains/`, so nothing else needs installing.

## Archives

| Release | Host | Archive | SHA-256 |
| --- | --- | --- | --- |
| [clang 22.1.8](https://github.com/stilt-lang/stilt-toolchains/releases/tag/clang-22.1.8) | macOS, Apple silicon | `clang-22.1.8-aarch64-macos.tar.gz`, 60 MB | `931abe51f03493b32544163f1e725321ed1c53aff219cd827c6d1c971dc0d773` |
| | Linux, x86-64 | `clang-22.1.8-x86_64-linux.tar.gz`, 91 MB | `e2832f51ddcc77f726c082474374ee973041b382f04f95459b333d258ef5d7bf` |
| | Linux, arm64 | `clang-22.1.8-aarch64-linux.tar.gz`, 118 MB | `71b0781b698a6379e6192da9d6d97f78bdd31b8b42d84390100b467434440588` |

Stilt 2027.1 names clang 22.1.8. The [clang 23.1.5](https://github.com/stilt-lang/stilt-toolchains/releases/tag/clang-23.1.5)
archives stay published but no release of `stilt` names them: LLVM 23.1.0 to 23.1.5 fold
`0.0f * (0.0f / d)` to `0.0f` when `d` comes out of a loop and may be zero, where the answer is
NaN (compare [llvm/llvm-project#241500](https://github.com/llvm/llvm-project/issues/241500)), which
breaks the promise that a release build means what the debug build means.

Each archive holds `bin/clang-22` and `bin/clang`, a link to it, and nothing else. A release
build compiles freestanding C, which includes no header, and links as a debug build does, so
it needs nothing more of LLVM. The Linux clang loads zlib, glibc 2.34 or later, and the
libstdc++ of GCC 12 or later (`GLIBCXX_3.4.30`). It doesn't run on systems with an older one of
these, such as Ubuntu 20.04 and Debian 11, or on Alpine, whose C library is musl: there,
`STILT_CC` names a clang of the machine for `stilt` to use. The macOS clang loads only the
system's libraries.

## Where they come from

`bin/clang-22` is the file of the same name in LLVM's own release binaries, unchanged:
`LLVM-22.1.8-macOS-ARM64.tar.xz`, `LLVM-22.1.8-Linux-X64.tar.xz` and `LLVM-22.1.8-Linux-ARM64.tar.xz` of the
[llvmorg-22.1.8 release](https://github.com/llvm/llvm-project/releases/tag/llvmorg-22.1.8).
Their build provenance was verified before repacking:

```sh
gh attestation verify LLVM-22.1.8-macOS-ARM64.tar.xz --repo llvm/llvm-project \
  --bundle LLVM-22.1.8-macOS-ARM64.tar.xz.jsonl
```

which names LLVM's `release-binaries.yml` workflow at `refs/tags/llvmorg-22.1.8` as the
builder. To check an archive against LLVM's, compare the hash of its `bin/clang-22` with the
same file of LLVM's tarball:

| Host | SHA-256 of `bin/clang-22` |
| --- | --- |
| aarch64-macos | `9a690e9e11d4774d63e95ed2ab90abd82889f949b0bb9db8208f7e18f278ea50` |
| x86_64-linux | `31a40dc31f3c15a47aa119aa148f339bf363d8f61202ddbdacbd3f24b71ba113` |
| aarch64-linux | `5e07dff5f5026f7cf09d49a68e04a8d27c662efe5fd5b63a2e569d65e0abc5cb` |

```sh
xz -dc LLVM-22.1.8-macOS-ARM64.tar.xz \
  | tar -xOf - LLVM-22.1.8-macOS-ARM64/bin/clang-22 | shasum -a 256
```

The archives are gzip-compressed POSIX tars written deterministically, entries in path order
with no owner, no time and modes 644 or 755, so the same files always give the same bytes.

## License

clang is part of the LLVM Project, under the Apache License 2.0 with LLVM Exceptions:
[LICENSE-LLVM.txt](LICENSE-LLVM.txt). The rest of this repository is under the MIT license or
the Apache License 2.0, at your option: [LICENSE-MIT](LICENSE-MIT), [LICENSE-APACHE](LICENSE-APACHE).
