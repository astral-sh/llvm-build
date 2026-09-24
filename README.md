# llvm-build

Build and package LLVM (Clang, lld, BOLT, and compiler runtimes) for
[python-build-standalone](https://github.com/astral-sh/python-build-standalone).

This project is based on similar work in
[indygreg/toolchain-tools](https://github.com/indygreg/toolchain-tools/tree/main/toolchain-bootstrap).


## Building

Run these commands from the root of this repository. Builds require substantial
disk space and time to compile LLVM from source.

For Linux, use Docker with Buildx and a host matching the requested architecture. 

```sh
# On a x86_64 host:
./build-linux-x86_64.sh

# On a aarch64 host:
./build-linux-aarch64.sh
```

Set `PARALLEL` to control build parallelism (default: 16). Archive timestamps
use the UTC date of `SOURCE_DATE_EPOCH`, which defaults to the latest Git commit's
timestamp.

On macOS, install `uv` and Xcode Command Line Tools, then run:

```sh
./build-macos.py
```

## Validating

Linux validation uses the corresponding builder image recorded in `build/` and
runs the relocated toolchain in a clean Jessie or Stretch container. It compiles
and runs C, C++ exceptions and atomics, shared and static C++ runtimes,
AddressSanitizer, and LTO examples.

```sh
./validate-linux.sh x86_64 build/llvm-gnu_only-x86_64-unknown-linux-gnu.tar.zst
./validate-linux.sh aarch64 build/llvm-gnu_only-aarch64-unknown-linux-gnu.tar.zst
```

On  macOS, extract the archive to a new location and run the smoke
tests, which cover C, C++, ThinLTO, and profile-guided optimization:

```sh
mkdir -p build/macos-smoke
uv run --no-project --python 3.14 python -m tarfile \
    --extract build/llvm-aarch64-apple-darwin.tar.zst build/macos-smoke
bash tests/smoke-macos.sh build/macos-smoke/llvm
```

## Releasing

Run the `Release` workflow manually with a release tag such as `20260923` and
the commit SHA of a successful `toolchain` build on `main`. It downloads that
build's LLVM tarballs, adds the LLVM version and release tag to their filenames,
and publishes them in a GitHub release. For example:
`llvm-23.1.2+20260923-gnu_only-x86_64-unknown-linux-gnu.tar.zst`.

Enable `dry-run` to download a preview of the renamed tarballs without publishing.

Release provisioning is managed in `astral-sh/github-policies`: `release-gate`
requires Full-time team approval without self-review or admin bypass, and the
`release-environment-gate` app then authorizes the publishing job's `release`
environment. Both environments are restricted to `main`.
