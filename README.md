<div align=right>Table of Contents↗️</div>

<h1 align=center><code>filepack</code></h1>

<div align=center>
  <a href=https://crates.io/crates/filepack>
    <img src=https://img.shields.io/crates/v/filepack.svg alt="crates.io version">
  </a>
  <a href=https://github.com/casey/filepack/actions>
    <img src=https://github.com/casey/filepack/actions/workflows/ci.yaml/badge.svg alt="build status">
  </a>
  <a href=https://github.com/casey/filepack/releases>
    <img src=https://img.shields.io/github/downloads/casey/filepack/total.svg alt=downloads>
  </a>
</div>

<br>

`filepack` is a command-line file hashing and verification utility written in
Rust.

It is an alternative to `.sfv` files and tools like `shasum`. Files are hashed
using [BLAKE3](https://github.com/BLAKE3-team/BLAKE3/), a fast, cryptographic
hash function.

A manifest named `filepack.json` containing the hashes of files in a directory
can be created with:

```sh
filepack create path/to/directory
```

Which will write the manifest to `path/to/directory/filepack.json`.

Files can later be verified with:

```
filepack verify path/to/directory
```

To protect against accidental or malicious corruption, as long as the manifest
has not been tampered with.

If you run `filepack` a lot, you might want to `alias fp=filepack`.

`filepack` is currently unstable: the interface and file format may change at
any time. Additionally, the code has not been extensively reviewed and should
be considered experimental.

Installation
------------

`filepack` is written in [Rust](https://www.rust-lang.org/) and can be built
from source and installed from a checked-out copy of this repo with:

```sh
cargo install --path .
```

Or from [crates.io](https://crates.io/crates/filepack) with:

```sh
cargo install filepack
```

See [rustup.rs](https://rustup.rs/) for installation instructions for Rust.

### Pre-Built Binaries

Pre-built binaries for Linux, MacOS, and Windows can be found on
[the releases page](https://github.com/casey/filepack/releases).

You can use the following command on Linux, MacOS, or Windows to download the
latest release, just replace `DEST` with the directory where you'd like to put
`filepack`:

```sh
curl --proto '=https' --tlsv1.2 -sSf https://filepack.com/install.sh | bash -s -- --to DEST
```

For example, to install `filepack` to `~/bin`:

```sh
# create ~/bin
mkdir -p ~/bin

# download and extract filepack to ~/bin/filepack
curl --proto '=https' --tlsv1.2 -sSf https://filepack.com/install.sh | bash -s -- --to ~/bin

# add `~/bin` to the paths that your shell searches for executables
# this line should be added to your shell's initialization file,
# e.g. `~/.bashrc` or `~/.zshrc`
export PATH="$PATH:$HOME/bin"

# filepack should now be executable
filepack --help
```

Note that `install.sh` may fail on GitHub Actions or in other environments
where many machines share IP addresses. `install.sh` calls GitHub APIs in order
to determine the latest version of `filepack` to install, and those API calls
are rate-limited on a per-IP basis. To make `install.sh` more reliable in such
circumstances, pass a specific tag to install with `--tag`.

Manifest format
---------------

`filepack` manifests are named `filepack.json`. `filepack create` and
`filepack verify` both expect the manifest to be in the root of the directory
containing the files to be verified.

The manifest contains a [JSON](https://www.json.org/json-en.html) object,
currently with a single key, `files`, whose value is an object mapping strings
containing paths to manifest entries. Manifest entries are objects with the key
`hash`, whose value is a hex-encoded BLAKE3 hash of the file, and `size`, whose
value is the integer length of the file in bytes.

Manifests are encoded as [UTF-8](https://en.wikipedia.org/wiki/UTF-8), so paths
must be valid Unicode.

Paths are `/`-separated, even on Windows, and may not contain backslashes.

Paths are relative, meaning that they cannot begin with a `/` or a Windows
drive prefix, such as `C:`, and cannot contain the path components `.` or
`..`, and may not end with a slash.

Filepack currently has no way of tracking empty directories, which are an error
when creating or verifying a manifest.

An example manifest for a directory containing the files `README.md` and
`src/main.c`, pretty-printed for legibility:

```json
{
  "files": {
    "README.md": {
      "hash": "5a9a6d96244ec398545fc0c98c2cb7ed52511b025c19e9ad1e3c1ef4ac8575ad",
      "size": 1573
    },
    "src/main.c": {
      "hash": "38abf296dc2a90f66f7870fe0ce584af3859668cf5140c7557a76786189dcf0f",
      "size": 4491
    }
  }
}
```

Lofty Ambitions
---------------

`filepack` has lofty ambitions!

- Definition of a "root" hash, likely just the hash of the manifest itself, so
  that as long as the root hash is received from a trusted source the manifest
  itself does not need to be trusted.

- Creation and verification of signatures over the root hash, so that
  developers and packagers can vouch for the correctness of the contents of a
  manifest, and users can verify that a manifest was signed by a trusted public
  key.

- Portability lints, so packagers can ensure that the files in a manifest can
  be used in other environments, for example case-insensitive and Windows file
  systems.

- Semantic, manchine-readable metadata about *what* a package is. For example,
  human-readable title, suggested filename-safe slug, description, or year of
  publication, to aid package indexing and search.

Suggestions for new features are most welcome!

Alternatives and Prior Art
--------------------------

`filepack` serves the same purpose as programs like `shasum`, which hash files
and output a text file containing file hashes and paths, which can later be
used with the same program to verify that the files have not changed.

They output hashes and paths one per line, separated by whitespace, and mainly
differ in which hash function they use.

Some examples, with links to implementations and the hash functions they use:

| Binary | Hash Function |
|---|---|
| [`cksfv`](https://zakalwe.fi/~shd/foss/cksfv/) | [CRC-32](https://en.wikipedia.org/wiki/Cyclic_redundancy_check) |
| [`shasum`](https://metacpan.org/pod/Digest::SHA) | [SHA-1](https://en.wikipedia.org/wiki/SHA-1) and [SHA-2](https://en.wikipedia.org/wiki/SHA-2) |
| [`sha3sum`](https://codeberg.org/maandree/sha3sum) | [SHA-3](https://en.wikipedia.org/wiki/SHA-3) |
| [`b2sum`](https://github.com/BLAKE2/BLAKE2) | [BLAKE2](https://en.wikipedia.org/wiki/BLAKE_(hash_function)#BLAKE2) |
| [`b3sum`](https://github.com/BLAKE3-team/BLAKE3) | [BLAKE3](https://en.wikipedia.org/wiki/BLAKE_(hash_function)#BLAKE3) |

CRC-32 is not a cryptographic hash function and cannot be used to detect
intentional modifications. Similarly, SHA-1 was thought to be a cryptographic
hash function, but is now known to be insecure.

`filepack` and `b3sum` both use BLAKE3, a fast, general-purpose cryptographic
hash function.

New Releases
------------

New releases of `filepack` are made frequently so that users quickly get access to
new features.

Release commit messages use the following template:

```
Release x.y.z

- Bump version: x.y.z → x.y.z
- Update changelog
- Update changelog contributor credits
- Update dependencies
```
