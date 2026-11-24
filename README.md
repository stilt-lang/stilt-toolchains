# Stilt toolchains

The clang that `stilt build --release` compiles C with. Each version of `stilt` names one
release of clang, and the URL and SHA-256 of an archive of it for each host. The first release
build on a machine downloads that archive, checks its hash, and unpacks it into
`~/.stilt/toolchains/`, so nothing else needs installing.

## Archives

| Release | Host | Archive | SHA-256 |
| --- | --- | --- | --- |
| [clang 23.1.5](https://github.com/stilt-lang/stilt-toolchains/releases/tag/clang-23.1.5) | macOS, Apple silicon | `clang-23.1.5-aarch64-macos.tar.gz`, 63 MB | `07510d2b4d7ecfe381afd73256d6007f2285927f15fa49a0fdd28d3f24aef3e9` |
| | Linux, x86-64 | `clang-23.1.5-x86_64-linux.tar.gz`, 95 MB | `008a14fc0261c04bb11ccac4f252be3814ea906d6f7b3ab24ca4b7fd5479fa40` |
| | Linux, arm64 | `clang-23.1.5-aarch64-linux.tar.gz`, 123 MB | `791ed84a6864c9d0323491654e4edbc23c8656993e859a3bbabd6339fea9151f` |

Each archive holds `bin/clang-23` and `bin/clang`, a link to it, and nothing else. A release
build compiles freestanding C, which includes no header, and links as a debug build does, so
it needs nothing more of LLVM. The Linux clang loads glibc 2.34 or later, libstdc++ and zlib,
which every current distribution has; the macOS clang loads only the system's libraries.

## Where they come from

`bin/clang-23` is the file of the same name in LLVM's own release binaries, unchanged:
`LLVM-23.1.5-macOS-ARM64.tar.zst`, `LLVM-23.1.5-Linux-X64.tar.zst` and `LLVM-23.1.5-Linux-ARM64.tar.zst` of the
[llvmorg-23.1.5 release](https://github.com/llvm/llvm-project/releases/tag/llvmorg-23.1.5).
Their build provenance was verified before repacking:

```sh
gh attestation verify LLVM-23.1.5-macOS-ARM64.tar.zst --repo llvm/llvm-project \
  --bundle LLVM-23.1.5-macOS-ARM64.tar.zst.jsonl
```

which names LLVM's `release-binaries.yml` workflow at `refs/tags/llvmorg-23.1.5` as the
builder. To check an archive against LLVM's, compare the hash of its `bin/clang-23` with the
same file of LLVM's tarball:

| Host | SHA-256 of `bin/clang-23` |
| --- | --- |
| aarch64-macos | `a22ab66d53cf70dfee8544eabdeb36f075bf942861cb8d59f98a3fadc7a65c26` |
| x86_64-linux | `2033aa32d783adec2a3d30a5d9229b60d9a7594ad73e19e87c3fd2b5ed21a9c2` |
| aarch64-linux | `5520f306dcff595b4e04a73ff8df1d332e3fd5d53040500295fc7778a363232a` |

```sh
zstd --long=30 -dc LLVM-23.1.5-macOS-ARM64.tar.zst \
  | tar -xOf - LLVM-23.1.5-macOS-ARM64/bin/clang-23 | shasum -a 256
```

The archives are gzip-compressed POSIX tars written deterministically, entries in path order
with no owner, no time and modes 644 or 755, so the same files always give the same bytes.

## License

clang is part of the LLVM Project, under the Apache License 2.0 with LLVM Exceptions:
[LICENSE-LLVM.txt](LICENSE-LLVM.txt). The rest of this repository is under the MIT license or
the Apache License 2.0, at your option: [LICENSE-MIT](LICENSE-MIT), [LICENSE-APACHE](LICENSE-APACHE).
