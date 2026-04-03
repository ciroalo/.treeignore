# Ignore File Resolution

**Project Name:** .treeignore  
**Project Manager:** @ciroalo  
**Last Revision Date:** April 3, 2026  
**Version:** 0.1  
**Status:** Stable (MVP subset)

This document describes how [tree-it](https://github.com/ciroalo/tree-it) resolves 
which ignore file to use when generating directory trees.

---

## Priority Order

`tree-it` resolves ignore rules using a strict priority:

| Priority | Source         | Capabilities                                    |
|----------|----------------|-------------------------------------------------|
| 1        | `.treeignore`  | Global exclusions, profiles, tags               |
| 2        | `.gitignore`   | Exclusion patterns only                         |
| 3        | None           | No exclusions — full directory tree is generated |

---

## Resolution Rules

1. If `.treeignore` exists in the target directory, it is used as the **sole configuration source**. The `gitignore` file is ignored completely, even if it also exists.

2. If `.treeignore` does not exist but `.gitignore` does, `.gitignore` is used for **exclusion patterns only**. Profiles and tags are not available in this mode.
 
3. If neither file exists, `tree-it` generates a complete directory tree with no exclusion rules applied.

---

## Lookup Location

Ignore files are resolved **only in the target directory** being analyzed. `tree-it` does not walk up parent directories or search globally.

### With an Explicit Path
 
```bash
tree-it ./my-project
```
 
Lookup occurs in:
 
```
./my-project/.treeignore
./my-project/.gitignore
```
 
### Without a Path (Default)
 
```bash
tree-it
```
 
The current working directory is used:
 
```
./.treeignore
./.gitignore
```
 
---

## `.gitignore` Fallback Behavior

When `.gitignore` is used as the fallback source:

- All non-comment, non-blank lines are treated as exclusion patterns

- Pattern matching follows the same rules as `.treeignore` patterns (see [matchign.md](matching.md))

- Profiles are **not available** - the `--profile` flag will produce an error

- Tags are **not available**

- Only the general tree is generated during default execution

---

## Profile Requirement
 
The `--profile` flag **requires** `.treeignore` to exist. If a user runs:
 
```bash
tree-it --profile tree_docs
```
 
and `.treeignore` is not found in the target directory, `tree-it` will print an error and exit with a non-zero status code — even if `.gitignore` exists.
 
---
 
## Summary
 
| Scenario                                 | Config source  | Profiles available | Tags available |
|------------------------------------------|----------------|-------------------|----------------|
| `.treeignore` exists                     | `.treeignore`  | Yes               | Parsed (not yet applied) |
| Only `.gitignore` exists                 | `.gitignore`   | No                | No             |
| Neither file exists                      | None           | No                | No             |
| `--profile` used without `.treeignore`   | Error          | —                 | —              |
 