# Profiles

**Project Name:** .treeignore  
**Project Manager:** @ciroalo  
**Last Revision Date:** April 3, 2026  
**Version:** 0.1  
**Status:** Stable (MVP subset)

This document describes the profile system in `.treeignore`

---

## What are Profiles?

Profiles let you define multiple filtered views of the same project tree. Each profile specifies additional exclusion patterns that apply on top of global exclusions when that profile is active.

This is useful when different or contexts need different views of the same project, for example,
a documentation view that hides test files, or a public view that hides internal tooling.

## Defining Profiles

A profile is a variable assignment in `.treeignore` whose name starts with `tree`:

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

Each quoted string in the list is an exclusion pattern. These patterns follow the same 
matching rules as global exclusions (see [matching.md](matching.md))

---

## Profile Naming

- Profile names must start with `tree` (e.g. `tree_docs`, `tree_api`, `tree_minimal`)

- Names are **case-insensitive** and normalized to lowercase internally.

- `tree_docs`, `Tree_docs`, and `TREE_DOCS` all refer to the same profile

---

## How Exclusions Combine

When a profile is active, its exclusion patterns are applied **in addition** to global exclusions. They do not replace them.

```
# Global exclusions — always applied
node_modules/
dist/
.git/

# Profile exclusions — added when tree_docs is active
tree_docs = [
    "tests/",
    "benches/"
]
```

Effective exclusions for the general tree:

```
node_modules/
dist/
.git/
```

Effective exclusions for tree_docs:

```
node_modules/
dist/
.git/
tests/
benches/
```

---

## Execution Modes

### Default Execution (No `--profile` flag)

```
tree-it
```

Generates **all** trees:

1. The **general tree** - applying only global exclusions

2. One tree for **each defined profile** - applying global + profile exclusions

Output labels each tree:

```
[general]
project/
├── src/
├── tests/
└── Cargo.toml

[tree_docs]
project/
├── src/
└── Cargo.toml
```

### Single Profile Execution

```bash
tree-it --profile tree_docs
```

Generates only the selected profile's tree. The general tree and other profiles are not included in the output.

---
 
## Error Handling
 
### Missing Profile
 
If the requested profile does not exist in `.treeignore`:
 
```bash
tree-it --profile tree_nonexistent
```
 
`tree-it` prints an error and exits with a non-zero status code.
 
### Profile Without `.treeignore`
 
The `--profile` flag requires `.treeignore` to exist. If `.treeignore` is not found:
 
```bash
tree-it --profile tree_docs
# Error: .treeignore not found
```
 
This applies even if `.gitignore` exists — profiles are a `.treeignore`-only feature.
 
### Only One Profile at a Time
 
Only a single profile may be selected per execution. Passing multiple `--profile` flags is not supported in the MVP.

---

## Profile Tags

Profiles may have associated tags that override or extend global tags when the profile is active. Profile tag variables follow the naming convention `<profile_name>_tags`:

```
tree_docs_tags = [
    "--level=2"
]
```

See [tags.md](tags.md) for details.

> **Note:** Tags are parsed in the current MVP but not yet applied 

