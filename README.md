# .treeignore

![treeignore-banner](docs/imgs/banner.svg)

`.treeignore` is a configuration file format for [tree-it](https://github.com/ciroalo/tree-it), a CLI tool that generates visual directory trees for documentation and project visualization.

A `.treeignore` file lets you control what appears in your project trees by defining exclusion rules, reusable profiles, and configuration tags - all from a single file placed in your projecrt root.

---

## Table of Contents


---

## Quick Start

Create a `.treeignore` file in your project root:

```
node_modules/
dist/
.git/
target/

tree_docs = [
    "tests/",
    "benches/"
]

tree_public = [
    "internal/",
    ".env"
]
```

Then run `tree-it`:

```bash
# generates the general tree + all profile trees
tree-it

# generates only the tree_docs profile
tree-it --profile tree_docs
```

---

## How it Works

`.treeignore` serves two purposes:

1. **Ignore file** - lines outside variable assignments are exclusion patterns that apply to every tree.

2. **Configuration source** - variable assignments define profiles and tags that control how different tree views are generated

When `tree-it` runs, it looks for `.treeignore` in the target directory. If found, it uses it as the sole configuration source (`.gitignore` is ignored). If `.treeignore` is absent, `tree-it` falls back to `.gitignore` for exclusion rules only. If neither file exists, the full directory tree is generated.

---

## File Format

A `.treeignore` file is a plain text file with three types of content:

### Global Exclusions

Any line that is not a variable assignment and not a comment is treated as a global **exclusion pattern**. These patterns apply to every tree output, the general tree and all profile trees.

```
# Dependencies
node_modules/
vendor/

# Build output
dist/
build/
target/

# Version control
.git/

# Specific files
*.log
```

Blank lines and lines starting with `#` are ignored.

### Profiles

Profiles define additional exclusion patterns that apply on top of global exclusions when a specific view of the project is requested.

A profile is a variable whose name starts with `tree`:

```
tree_docs = [
    "tests/",
    ".github/",
    "benches/"
]

tree_public = [
    "internal/",
    ".env",
    "scripts/"
]
```

**Syntax rules**

- The variable name must start with `tree` (e.g. `tree_docs`, `tree_api`, `tree_public`)

- The value is a bracket-delimited list of quoted strings.

- Each string is an exclusion pattern following the same matching rules as global exclusions.

- Profile names are case-insensitive and normalized to lowercase internally.

**Behavior**

- Running `tree-it` without `--profile` generates the general tree (global exclusions only) and one tree for each defined profile (global + profile exclusions).

- Running `tree-it --profile tree_docs` generates only that profile's tree.

### Tags

Tags are CLI-style options defined inside `.treeignore` that influence how trees are generated.

**Global tags** apply to all trees unless overridden:

```
tags = [
    "--level=3",
    "--dirsfirst"
]
```

**Profile tags** apply when a specific profile is active. They merge with global tags and override conflicting values:

```
tree_docs_tags = [
    "--level=2",
    "--ascii"
]
```

Profile tag variables follow the naming convention `<profile_name>_tags`

**Preference order:**

```
CLI flags > profile tags > global tags
```

> **Note:** Tags are parsed in the current MVP but not yet applied. Tag behavior is planned for a future version.

---

## Pattern Matching

`.treeignore` uses an in-house pattern matcher inspired by `.gitignore` semantics. 
It supports a defined subset of pattern features.

### Supported Patterns

| Pattern   | Description                              | Example           |
|-----------|------------------------------------------|-------------------|
| `*`       | Matches any sequence within a path segment | `*.log`          |
| `?`       | Matches any single character             | `file?.txt`       |
| `**`      | Matches zero or more directories         | `**/temp`         |
| Trailing `/` | Matches directories only              | `build/`          |
| Leading `/`  | Anchors pattern to the root           | `/target`         |

### How Matching Works 

- Patterns are matched against **normalized relative paths** from the analyzed root.

- Path separators are normalized to `/` on all platforms

- Patterns match **anywhere in the tree** by default - a pattern like `temp/` will match any directory named `temp` at any depth

- A leading `/` **anchors** the pattern to the root of the analyzed tree - `/target` only matches `target` at the root, not deeper

- Leading `/` and trailing `/` can be combined - `/build/` matches only a directory named `build` at the root

- When a directory is matched, it is **pruned entirely** - it does not appear in the output and its contents are not traversed

### Not Supported (MVP)

- Negation / re-inclusion patterns (`!file.txt`)

- Advanced escaping semantics

- Full `.gitignore` edge-case compatibility

For the complete pattern matching specification, see [matching.md](docs/matching.md)

---

## Examples

See the [examples](examples/) directory for sample `.treeignore` files:

- `basic.treeignore` - simple exclusions only

- `profiles.treeignore` - multiple profile definitions

- `full.treeignore` - exclusions, profiles, and tags combined

---

## Current Status

The `.treeignore` format is at **v0.1** and is consumed by [tree-it](https://github.com/ciroalo/tree-it) (MVP)

**Stable in V0.1:**

- Global exclusion patterns

- Profile definitions

- Pattern matching (documented subset)

- Ignore file resolution priority

**Parsed but not yet applied:**

- Global tags

- Profile tags

- Tags precedence behavior

**Planned:**

- Extended matching semantics

- Image output

---

## Documentation
 
| Document | Description |
|----------|-------------|
| [docs/format.md](docs/format.md) | Full file format specification |
| [docs/matching.md](docs/matching.md) | Pattern matching rules and examples |
| [docs/resolution.md](docs/resolution.md) | Ignore file resolution strategy |
| [docs/profiles.md](docs/profiles.md) | Profile system reference |
| [docs/tags.md](docs/tags.md) | Tag system reference |
 
---
 
## Related
 
- [tree-it](https://github.com/ciroalo/tree-it) — the CLI tool that consumes `.treeignore` files
 
---
 
## License
 
MIT License
 
## Author
 
Ciro Alonso
 