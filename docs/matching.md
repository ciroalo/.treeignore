# Pattern Matching

**Version:** 0.1  
**Status:** Stable (MVP subset) 

This document defines the pattern matching rules used by `.treeignore` and `.gitignore` fallback in [tree-it](https://github.com/ciroalo/tree-it)


## Overview

`tree-it` uses an in-house pattern matcher inspired by `.gitignore` semantics. For the MVP, it supports a **documented subset** of pattern features rather than full `.gitignore` compatibility.

All patterns, whether from global exclusions or profile definitions, follow the same matching rules described here.


## Path Normalization

Before any matching occurs, candidate paths are normalized:

1. Paths are made relative to the analyzed root directory.

2. All path separators are normalized to `/`, regardless of platform.

3. Matching uses these normalized paths consistently across macOS, Linux and Windows.

For example, given `tree-it ./my-project`, a file at `./my-project/src/main.rs` is matched against the relative path `src/main.rs`.


## Supported Patterns

### `*` - Single-Segment Wildcard

Matches any sequence of characters **within a single path segment** (i.e. it does not cross `/` boundaries)

| Pattern    | Matches              | Does Not Match       |
|------------|----------------------|----------------------|
| `*.log`    | `error.log`          | `logs/error.log`     |
| `src/*.rs` | `src/main.rs`        | `src/util/helpers.rs`|
| `temp*`    | `temporary`, `temp1` | `dir/temporary`      |

### `?` - Single Character Wildcard

Matches exactly **one character**, excluding `/`

| Pattern     | Matches        | Does Not Match  |
|-------------|----------------|-----------------|
| `file?.txt` | `file1.txt`    | `file10.txt`    |
| `?.rs`      | `a.rs`         | `ab.rs`         |

### `**` - Multi-Directory Wildcard

Matches **zero or more directories**. The behavior depends on its position in the pattern:

**Leading** `**/` - matches at any depth:

| Pattern       | Matches                             |
|---------------|-------------------------------------|
| `**/temp`     | `temp`, `src/temp`, `a/b/c/temp`    |
| `**/*.log`    | `error.log`, `logs/error.log`       |

**Trailing** `/**` - matches everything inside a directory:

| Pattern            | Matches                               |
|--------------------|---------------------------------------|
| `src/**/test.rs`   | `src/test.rs`, `src/a/b/test.rs`      |

**Middle** `/**/` - matches zero or more intermediate directories

| Pattern            | Matches                               |
|--------------------|---------------------------------------|
| `src/**/test.rs`   | `src/test.rs`, `src/a/b/test.rs`      |

### Trailing `/` - Directory-Only Matching

A pattern ending with `/` matches **directories only**, not files:

| Pattern   | Matches               | Does Not Match       |
|-----------|-----------------------|----------------------|
| `build/`  | `build/` (directory)  | `build` (file)       |
| `temp/`   | `temp/` (directory)   | `temp` (file)        |

When a directory matches, it is **pruned entirely**: it does not appear in the output and its contents are not traversed.

### Leading `/` - Root-Anchored Matching

A pattern starting with `/` is **anchored to the root** of the analyzed tree. It only matches entries at the root level, not deeper in the hierarchy:

| Pattern    | Matches                        | Does Not Match              |
|------------|--------------------------------|-----------------------------|
| `/target`  | `target` (file or dir at root) | `src/target`, `a/b/target`  |
| `/target/` | `target/` (directory at root)  | `src/target/`, `a/target/`  |
| `/dist`    | `dist` (file or dir at root)   | `lib/dist`                  |

Leading `/` can be combined with trailing `/` to anchor a directory-only pattern to the root. Without a trailing `/`, the anchored pattern matches both files and directories at the root.

This is useful when a common name like `target` or `build` exists at multiple levels but you only want to exclude the one at the project root.


## Matching Scope

By default, patterns match **anywhere in the tree**. A pattern does not need to specify the full path from the root.

For example the pattern `temp/` matches any directory named `temp` at any depth in the analyzed tree.

To restrict a pattern to the root level only, prefix it with `/`. For example, `/temp/` matches only a `temp` directory at the root of the analyzed tree.


## Matching Behavior

- **First-match-wins:** the first pattern that matches a given path determines whether it is excluded.

- **Pruning:** when a directory matches any exclusion pattern, the entire subtree is skipped during traversal. No children of that directory will appear in the output.

- **Global + profile:** when a profile is active, global exclusion patterns are evaluated first, followed by profile-specific patterns.


## Not supported (MVP)

The following features are **not supported** in the current version:

| Feature                    | Status         |
|----------------------------|----------------|
| Negation (`!pattern`)      | Not supported  |
| Advanced escaping (`\`)    | Not supported  |
| Full `.gitignore` compatibility | Not supported |
| Character classes (`[abc]`) | Not supported |

These may be added in future versions.


## Examples

### Basic file exclusion 

```
*.log
*.tmp
*.bak
```

Excludes all `.log`, `.tmp` and `.bak` files at any depth.

### Directory Exclusion

```
node_modules/
dist/
.git/
target/
```

Excludes these directories and all their contents, whether they appear in the tree.

### Nested Wildcards

```
**/fixtures/**
```

Excludes everything inside any `fixtures` directory at any depth

### Mixed Patterns 

```
# Directories

build/
.cache/

# Files by extension
*.o
*.pyc

# Specific nested pattern
src/**/generated/*.rs
```

### Root-Anchored Patterns

```
# Only exclude target/ at the project root, not nested ones
/target
/target/

# Only exclude the root-level dist, not lib/dist
/dist

# Combine with directory-only matching
/build/
```