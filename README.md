# ast-grep

Container images with [ast-grep](https://github.com/ast-grep/ast-grep), a tool that searches, lints and rewrites code with patterns written as code. ast-grep parses each file with tree-sitter and matches the pattern against the syntax tree, so `console.log($A)` finds calls with one argument whatever their formatting, and skips the same text in strings and comments. Rules in YAML files turn patterns into a linter for a whole project. The images build ast-grep from the source of each ast-grep release with a pinned Rust release, with every language ast-grep supports. They are built on Ubuntu and Alpine for `linux/amd64` and `linux/arm64`, and rebuilt when ast-grep publishes a release and when the base image changes.

This is an unofficial build, not affiliated with or endorsed by the ast-grep project. Report problems with the image in this repository and problems with ast-grep itself in the [ast-grep issue tracker](https://github.com/ast-grep/ast-grep/issues).

## Quick start

Find every `console.log` call in the TypeScript files under `src/`:

```sh
docker run --rm --user "$(id -u):$(id -g)" -v "$PWD:/work" ghcr.io/randomcontainers/ast-grep run -p 'console.log($$$ARGS)' -l ts src/
```

A pattern is code in the language that `-l` names. `$NAME` matches one syntax node and `$$$NAME` any number of them, and a name used twice must match the same code both times. Without `-l`, ast-grep takes the language of each file from its extension. Without paths it searches the working directory, `/work`. `run` exits with 0 when it found a match and with 1 when it found none. Patterns hold `$`, so keep them in single quotes.

`-r` gives the replacement, which can use the names from the pattern. `run` then prints each change as a diff, and `-U` (`--update-all`) writes all of them into the files:

```sh
docker run --rm --user "$(id -u):$(id -g)" -v "$PWD:/work" ghcr.io/randomcontainers/ast-grep run -p 'var $A = $B;' -r 'let $A = $B;' -l js -U src/
```

`-i` asks about each change instead, and needs a terminal: add `-it` to `docker run`. `--json` prints the matches as a JSON array, `--json=stream` as one object per line. `--stdin` reads code from standard input, with `-l` and without mounting anything:

```sh
echo 'console.log(1)' | docker run --rm -i ghcr.io/randomcontainers/ast-grep run -p 'console.log($A)' -l js --stdin
```

Without arguments the image prints ast-grep's help.

## Projects and rules

`ast-grep scan` checks a project against rules kept in YAML files. `sgconfig.yml` at the root of the project names the directories they are in:

```yaml
ruleDirs:
  - rules
testConfigs:
  - testDir: rule-tests
```

A rule in `rules/no-console.yml`:

```yaml
id: no-console
message: Use the logger instead of console.log
severity: error
language: TypeScript
rule:
  pattern: console.log($$$ARGS)
```

```sh
docker run --rm --user "$(id -u):$(id -g)" -v "$PWD:/work" ghcr.io/randomcontainers/ast-grep scan
```

`scan` exits with 1 when a rule with `severity: error` matches, and with 0 when only rules of a lower severity do. `--format sarif` writes the results as SARIF for GitHub code scanning, `--format github` as annotations for a GitHub Actions log, and `--json` as JSON. `ast-grep test --skip-snapshot-tests` checks each rule against the `valid` and `invalid` examples in the test directories. `ast-grep new` creates `sgconfig.yml` and new rules; it asks questions, so run it with `-it`, or pass `--yes` to take the defaults.

ast-grep looks for `sgconfig.yml` in the working directory and its parents. In the container the parents of `/work` are the image's own directories, so mount the root of the project, or pass the file with `-c`. The [rule reference](https://ast-grep.github.io/reference/yaml.html) lists every key.

In GitLab CI, clear the entrypoint:

```yaml
ast-grep:
  image:
    name: ghcr.io/randomcontainers/ast-grep:<version>
    entrypoint: [""]
  script:
    - ast-grep scan
```

Replace `<version>` with the newest exact version tag (see [Tags](#tags)), and update it by hand after each ast-grep release: a new release can parse code differently and change what your rules match.

## Custom languages

`customLanguages` in `sgconfig.yml` adds languages from tree-sitter grammars compiled as shared libraries. ast-grep ignores them unless you pass `--custom-languages=allow`. `--custom-languages=trust` remembers the project instead, but it needs a terminal and writes to a directory that does not outlive the container, so use `allow` in the image. A library must be built for the C library of the image that loads it: glibc for the Ubuntu images and musl for the Alpine images.

## What is in the image

- `ast-grep` in `/usr/local/bin`, built from the release tag with upstream's release profile, which uses link-time optimization. `sg`, the crate's other binary, is a deprecated alias that runs `ast-grep` and is not included.
- Every language built into ast-grep: Bash, C, C++, C#, CSS, Dart, Elixir, Go, Haskell, HCL, HTML, Java, JavaScript, JSON, Kotlin, Lua, Markdown, Nix, PHP, Python, Ruby, Rust, Scala, Solidity, Swift, TypeScript, TSX, YAML and Zig.

ast-grep is linked against the distro's C library and libgcc_s: glibc on Ubuntu, musl on Alpine. Copying `/usr/local/bin/ast-grep` into another image with `COPY --from` works only on a base with the same C library, at the same or a newer version, and libgcc_s.

Upstream's `.cargo/config.toml` names cross compilers as the linkers for `aarch64-unknown-linux-gnu`, `aarch64-unknown-linux-musl` and `armv7-unknown-linux-gnueabihf`. The images are built natively on each architecture and link with the distro's `cc` instead, and armv7 is not built. The file's musl setting, `-C target-feature=-crt-static`, is the flag the Alpine image is built with anyway.

`/usr/local/share/randomcontainers/ast-grep/buildinfo` has the commit, the Rust release with the URL and SHA-256 of each archive it was installed from, the output of `rustc -vV`, the cargo-auditable version, the version ast-grep reports, and every crate the binary was built from, with its version, whether it is linked in or used only during the build, and its license.

## Default or slim

ast-grep's default image adds no other tools, so `latest` and `slim` are the same image, with the contents listed above. Use `latest` to run it and the `slim` tags as a base for your own image.

## Tags

`<version>` is an ast-grep release such as `0.51.0`. `<minor>` and `<major>` are its shorter forms, `0.51` and `0`, and follow the newest release in that series. While ast-grep is at 0.x, `<major>` moves to every release. Each row lists the default tag and its `slim` twin, which point to the same image.

| Tags | Base |
|---|---|
| `latest`, `slim` | Ubuntu |
| `<version>`, `<version>-slim` | Ubuntu |
| `<minor>`, `<minor>-slim`, `<major>`, `<major>-slim` | Ubuntu |
| `ubuntu`, `slim-ubuntu` | Ubuntu |
| `<version>-ubuntu`, `<version>-slim-ubuntu` | Ubuntu |
| `<minor>-ubuntu`, `<minor>-slim-ubuntu`, `<major>-ubuntu`, `<major>-slim-ubuntu` | Ubuntu |
| `<version>-ubuntu26.04`, `<version>-slim-ubuntu26.04` | Ubuntu 26.04 |
| `alpine`, `slim-alpine` | Alpine |
| `<version>-alpine`, `<version>-slim-alpine` | Alpine |
| `<minor>-alpine`, `<minor>-slim-alpine`, `<major>-alpine`, `<major>-slim-alpine` | Alpine |
| `<version>-alpine3.24`, `<version>-slim-alpine3.24` | Alpine 3.24 |

The images are currently built on Ubuntu 26.04 and Alpine 3.24. Tags without a distro version move to the next distro release when the project does; tags ending in `ubuntu26.04` or `alpine3.24` stay on that release and are no longer rebuilt once the project moves to the next one. Every tag of the current ast-grep version, including the exact version, is rebuilt in place (see [Updates](#updates)), so pin a digest when you need the same bytes every time. In CI, pin at least the exact version.

## Platforms

`linux/amd64` and `linux/arm64`, for both Ubuntu and Alpine. Each image contains ast-grep built for its own architecture and is built natively on GitHub-hosted runners, without emulation.

## Files and permissions

The working directory is `/work`. The image runs as UID 1000, and any other UID works too: `HOME` is then `/`, and caches go to `/cache`, which anyone can write to. ast-grep only reads files unless you rewrite them with `-U` or `-i` or create files with `ast-grep new`, so for searches and scans the mount can be read-only (`-v "$PWD:/work:ro"`). It can read and change only the files its UID has access to, so run it as your own user:

| Runtime | Flag |
|---|---|
| Docker on Linux (rootful), GitHub Actions | `--user "$(id -u):$(id -g)"` |
| Rootless Podman | `--userns=keep-id` |
| Rootless Docker | `--user 0:0` (root in the container is your user on the host) |
| Docker Desktop on macOS or Windows | none, file ownership is mapped for you |

## Extending the slim image

Use a `slim` tag as the base for your own image. `slim`, `slim-ubuntu` and `slim-alpine` move to each new ast-grep release and are rebuilt when the base image changes. Switch to root to install more, then back. This adds `git`, for a script that scans only the files changed on a branch:

```dockerfile
FROM ghcr.io/randomcontainers/ast-grep:slim-ubuntu@sha256:...
USER root
RUN apt-get update \
 && apt-get install -y --no-install-recommends git \
 && rm -rf /var/lib/apt/lists/*
USER 1000:1000
```

On Alpine, start from `slim-alpine` and use `apk add --no-cache git`. The entrypoint is `["tini", "--", "ast-grep"]`; set your own `ENTRYPOINT` if your image runs a script. To pick up new ast-grep releases and base image fixes, let Dependabot or Renovate update the digest in your `FROM` line.

Everything the image adds is under `/usr/local`. `/usr/local/share/randomcontainers/ast-grep/` holds the version, the source URLs, the build details, the license files, the source of the one MPL-2.0 crate in `crate-sources/`, and `runtime-deps`, the list of distro packages ast-grep needs at run time. It is empty on Ubuntu, and on Alpine it names musl and libgcc_s by their `so:` names.

## Verifying

Each image has a build provenance attestation from this repository's GitHub Actions run, signed by the shared build workflow in `randomcontainers/ci`:

```sh
gh attestation verify oci://ghcr.io/randomcontainers/ast-grep:latest \
  --repo randomcontainers/ast-grep --signer-repo randomcontainers/ci
```

Each platform image also carries an SPDX SBOM that lists every distro package with its version:

```sh
docker buildx imagetools inspect ghcr.io/randomcontainers/ast-grep:latest --format '{{ json .SBOM }}'
```

ast-grep is built with [cargo-auditable](https://github.com/rust-secure-code/cargo-auditable), which stores the list of crates compiled into it in the binary's `.dep-v0` section. Syft, Grype and Trivy read that list, so SBOM tools and image scanners see those crates. The Rust standard library is not in the list, and scanners do not report its advisories; `buildinfo` names the Rust release that built ast-grep, and [Updates](#updates) says when it moves. The C code of tree-sitter and of the language grammars is compiled from the crates that carry it, which the list includes. Follow ast-grep's [security advisories](https://github.com/ast-grep/ast-grep/security) as well.

ast-grep publishes no source tarball, so each version is pinned to the commit of its tag. The build clones the `<version>` tag of [ast-grep/ast-grep](https://github.com/ast-grep/ast-grep) and stops unless the tag points to the commit recorded in `package.yml`. cargo downloads the crates in the tag's `Cargo.lock` and checks each against the checksum there, and the build that follows has no network access. It also stops when upstream's `.cargo/config.toml`, which applies to the build, differs from the SHA-256 pinned in `cargo-config.sha256`.

Rust comes from static.rust-lang.org. The release manifest named in `package.yml` is checked against the SHA-256 recorded there and against its signature, made with the Rust release key (fingerprint `108F 6620 5EAE B0AA A8DD 5E1C 85AB 96E6 FA1B E5FE`, kept in [keys/rust.gpg](keys/rust.gpg)). Each archive installed from it is checked against the SHA-256 that the manifest lists. cargo-auditable, which runs only during the build, is compiled from its crates.io crate, checked against the SHA-256 in `package.yml`, with the crates listed in its own `Cargo.lock`.

## Updates

The project checks the releases of [ast-grep/ast-grep](https://github.com/ast-grep/ast-grep/releases) every 15 minutes. A release is picked up once it is 24 hours old. The new version and the commit of its tag are then committed to `package.yml` and the images are rebuilt. Only the newest release is built; tags of older versions stay as they were last built.

The Rust release that builds ast-grep follows Rust's stable channel. Each new stable release is committed to `package.yml` once it is 7 days old, and the images are rebuilt with it. cargo-auditable is pinned by hand. An ast-grep release that needs a newer Rust than the pinned one does not build until the pin moves. A release also waits for a person when it adds a crate under a license that the image's label does not cover, adds a crate without a license file, adds a crate that builds or links native code (one with a `links` key), drops a crate whose license text this repository keeps or changes its license (see [Licenses](#licenses)), changes the version of Inflector, schemafy_core or schemafy_lib, moves the ICU license inside the tree-sitter crate, or changes upstream's `.cargo/config.toml`.

The images of the current version are also rebuilt when the Ubuntu or Alpine base image changes and at least every 7 days, so distro security fixes reach the current tags.

## Building

```sh
docker build -f Dockerfile.ubuntu --target slim \
  --build-arg VERSION=<version> \
  --build-arg SOURCE_COMMIT=<commit from package.yml> \
  --build-arg RUST_TOOLCHAIN_VERSION=<rust-toolchain version from package.yml> \
  --build-arg RUST_TOOLCHAIN_SHA256=<rust-toolchain sha256 from package.yml> \
  --build-arg CARGO_AUDITABLE_VERSION=<cargo-auditable version from package.yml> \
  --build-arg CARGO_AUDITABLE_SHA256=<cargo-auditable sha256 from package.yml> \
  -t ast-grep:local .
```

Use `Dockerfile.alpine`, with the same arguments, for the Alpine image. The build downloads 115 to 135 MB of Rust archives on Ubuntu and 165 to 170 MB on Alpine, and the crates in `Cargo.lock`. `--build-arg JOBS=<n>` limits the number of crates cargo compiles in parallel. The grammars are large generated C files, and the link-time optimization of the final binary runs mostly on one core.

## Licenses

ast-grep is available under the MIT license. The binary also contains the Rust crates ast-grep depends on. Most of them are under `MIT OR Apache-2.0` or MIT, and the others offer a choice of licenses that includes MIT, except these: codespan-reporting, similar, sync_wrapper, tree-sitter-elixir and tree-sitter-hcl are Apache-2.0 only, encoding_rs includes WHATWG data under BSD-3-Clause, libloading is under ISC, borrow-or-share under MIT-0 and option-ext under the Mozilla Public License 2.0 (MPL-2.0). The tree-sitter crate compiles three headers from ICU, under the license file that ships with them: the Unicode license of ICU 58 and later (Unicode-DFS-2016), and the ICU License for their IBM code from ICU 57 and earlier. The Rust standard library and the crates it uses are linked in too. They are under `MIT OR Apache-2.0` or another choice that includes MIT, except compiler-builtins, which is under `MIT AND Apache-2.0 WITH LLVM-exception`, and the Unicode data tables in core, which are under Unicode-3.0.

The image's license label is `MIT AND Apache-2.0 AND BSD-3-Clause AND ISC AND MIT-0 AND MPL-2.0 AND Unicode-3.0 AND Unicode-DFS-2016 AND ICU AND Apache-2.0 WITH LLVM-exception`, with MIT chosen wherever a crate offers it. The build stops when a crate linked into ast-grep has a license that this label does not cover. Crates used only while ast-grep is built are not in the binary, so the label leaves out their licenses, such as BSD-2-Clause for Inflector.

The image includes the source of option-ext, as the MPL-2.0 asks: its crate from crates.io, checked against `Cargo.lock`, is in `/usr/local/share/randomcontainers/ast-grep/crate-sources/`, and `source` lists its URL.

The license files are in `/usr/local/share/randomcontainers/ast-grep/licenses/`: ast-grep's `LICENSE` from the release tag; `crates/<crate>-<version>/` with the license files of every crate that the binary records, including those used only while it is built, and `crates/CRATES.tsv`, which lists each of them with its license; the ICU license in `crates/tree-sitter-<version>/src/unicode/LICENSE`; and `rust/` with the license files of rust-lang/rust, the standard library and the crates it vendors, from the `rust-src` archive of the Rust release that built ast-grep (all of them except LLVM's libunwind, which the dynamically linked binary does not contain). Fifteen crates publish no license file: the grammars for C++, HTML, Java, JSON, Lua, Markdown, Ruby, TypeScript and Zig, inquire, crokey-proc_macros, lazy-regex-proc_macros, Inflector, schemafy_core and schemafy_lib. This repository keeps their license texts in [licenses](licenses), and the build copies them into the image. The Ubuntu and Alpine packages in the image keep their own licenses.

The source for each image:

- ast-grep: built from the `<version>` tag of [ast-grep/ast-grep](https://github.com/ast-grep/ast-grep). `/usr/local/share/randomcontainers/ast-grep/source` names the commit of that tag, the Rust release manifest, the `rust-src` archive with the source of the Rust standard library, the cargo-auditable crate and the option-ext crate. The crates are on [crates.io](https://crates.io/) at the versions in that commit's `Cargo.lock`, and `buildinfo` lists those ast-grep was built from.
- Build scripts: this repository at the commit in the image's `org.opencontainers.image.revision` label.
- Ubuntu packages: the source packages on [Launchpad](https://launchpad.net/ubuntu) for the versions listed in the SBOM. `apt-get source <package>=<version>` fetches a version that is still in the Ubuntu archive.
- Alpine packages: Alpine has no source packages. For the versions listed in the SBOM, the source is the APKBUILD and patches in [aports](https://gitlab.alpinelinux.org/alpine/aports/-/tree/3.24-stable), branch `3.24-stable`, and the archives on [distfiles.alpinelinux.org](https://distfiles.alpinelinux.org/distfiles/v3.24/).

The files in this repository are available under the MIT license, see [LICENSE](LICENSE), except the license texts in `licenses/`, which keep their own terms.

## Requesting a tool

To suggest another tool, use the [Request a tool](https://github.com/randomcontainers/.github/issues/new?template=tool-request.yml) form.
