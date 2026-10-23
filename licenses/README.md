# Extra license files

Fifteen crates that ast-grep is built from publish no license text. Each directory in `crates/` holds the text for one of them, and `scripts/cargo-licenses.py` adds it to the crate's own files in the image. The build stops when the binary records a crate without any license file, and when a directory here matches no crate it records.

A directory named with a version, `crates/<crate>-<version>/`, covers only that version, so the build also stops when the crate moves to another version, and the text is checked again before the directory is renamed. A directory without a version, `crates/<crate>/`, covers every version. Its `EXPRESSION` file holds the crate's license expression, and the build stops when the crate's expression changes. The grammars and the other crates that change often have directories without a version.

| Files | Source |
|---|---|
| `crates/tree-sitter-cpp/LICENSE` | `LICENSE` of [tree-sitter/tree-sitter-cpp](https://github.com/tree-sitter/tree-sitter-cpp) at v0.23.4 |
| `crates/tree-sitter-html/LICENSE` | `LICENSE` of [tree-sitter/tree-sitter-html](https://github.com/tree-sitter/tree-sitter-html) at v0.23.2 |
| `crates/tree-sitter-java/LICENSE` | `LICENSE` of [tree-sitter/tree-sitter-java](https://github.com/tree-sitter/tree-sitter-java) at v0.23.5 |
| `crates/tree-sitter-json/LICENSE` | `LICENSE` of [tree-sitter/tree-sitter-json](https://github.com/tree-sitter/tree-sitter-json) at v0.24.8 |
| `crates/tree-sitter-lua/LICENSE.md` | `LICENSE.md` of [tree-sitter-grammars/tree-sitter-lua](https://github.com/tree-sitter-grammars/tree-sitter-lua) at v0.5.0 |
| `crates/tree-sitter-md/LICENSE` | `LICENSE` of [tree-sitter-grammars/tree-sitter-markdown](https://github.com/tree-sitter-grammars/tree-sitter-markdown) at v0.5.3 |
| `crates/tree-sitter-ruby/LICENSE` | `LICENSE` of [tree-sitter/tree-sitter-ruby](https://github.com/tree-sitter/tree-sitter-ruby) at v0.23.1 |
| `crates/tree-sitter-typescript/LICENSE` | `LICENSE` of [tree-sitter/tree-sitter-typescript](https://github.com/tree-sitter/tree-sitter-typescript) at v0.23.2 |
| `crates/tree-sitter-zig/LICENSE` | `LICENSE` of [tree-sitter-grammars/tree-sitter-zig](https://github.com/tree-sitter-grammars/tree-sitter-zig) at v1.1.2 |
| `crates/inquire/LICENSE` | `LICENSE` of [mikaelmello/inquire](https://github.com/mikaelmello/inquire) at v0.9.4 |
| `crates/crokey-proc_macros/LICENSE` | `LICENSE` of [Canop/crokey](https://github.com/Canop/crokey) at commit a183b147, from which crokey-proc_macros 1.5.0 was published |
| `crates/lazy-regex-proc_macros/LICENSE` | `LICENSE` of [Canop/lazy-regex](https://github.com/Canop/lazy-regex) at commit 39a459c0, from which lazy-regex-proc_macros 3.6.1 was published |
| `crates/Inflector-0.11.4/LICENSE.md` | `LICENSE.md` of [whatisinternet/Inflector](https://github.com/whatisinternet/Inflector) at commit a4a95eac, from which Inflector 0.11.4 was published |
| `crates/schemafy_core-0.6.0/LICENSE`, `crates/schemafy_lib-0.6.0/LICENSE` | `LICENSE` of [Marwes/schemafy](https://github.com/Marwes/schemafy) at v0.6.0 |

Each file was taken from the commit that the crate's `.cargo_vcs_info.json` names, which for the tagged crates is the commit of that tag.

`crates/native-covered` lists the crates with a `links` key whose license files cover the native code they build. The tree-sitter crate compiles its C runtime, which includes three ICU headers; the line `tree-sitter: src/unicode/LICENSE` copies the ICU license that the crate ships next to them. tree-sitter-language builds no C code on Linux, and its own license covers it.
