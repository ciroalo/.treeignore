# .treeignore File Format

**Project Name:** .treeignore  
**Project Manager:** @ciroalo  
**Last Revision Date:** April 3, 2026  
**Version:** 0.1  
**Status:** Stable (MVP subset)

This document defines the complete syntax and semantics of the `.treeignore` file format.


## Overview

A `.treeignore` file is a plain text file that serves as both an ignore file and a configuration source for [tree-it](https://github.com/ciroalo/tree-it).

It supports three types of content:

1. **Global exclusion patterns** - lines outside variable assignments

2. **Variable assignment** - profiles and tags

3. **Comments and blank lines** - ignored during parsing


## Encoding

- Files must be UTF-8 encoded.

- Lines endings may be `\n` (Unix) or `\r\n` (Windows)


## Comments

Lines starting with `#` are comments and are ignored by the parser

```
# This is a comment
node_modules/
```

Inline comments are not supported. The `#` character is only treated as a comment marker when it is the first non-whitespace character on a line.


## Blank Lines

Blank lines (empty or white-space only) are ignored and have no effect.


## Global Exclusion Patterns

Any line that is not a comment, not blank, and not part of a variable assignment is treated as a global exclusion pattern

```
node_modules/
dist/
.git/
*.log
```

These patterns apply to **every** tree output - the general tree and all profile trees.
Pattern matching rules are defined in [matching.md](matching.md)


## Variable Assignments

Variable assignments use the following syntax: 

```
variable_name = [
    "value1",
    "value2"
]
```

**Syntax rules:**

- The variable name must be a valid identifier: lowercase letters, digits, and underscores

- The `=` sign separates the name from the value

- The value is a bracket-delimited list `[ ... ]`

- Each element is a double-quoted string

- Elements are separated by commas. A trailing comma after the last element is allowed.

- Whitespace around `=`, inside brackets, and around elements is ignored

**Compact form** is also valid:

```
tree_docs = ["tests/", ".github"]
```

### Variable Types

Variables are classified by their name:

| Name pattern         | Type         | Description                     |
|----------------------|--------------|---------------------------------|
| `tree_*`             | Profile      | Defines a tree profile          |
| `tags`               | Global tags  | CLI-style options for all trees |
| `<profile>_tags`     | Profile tags | CLI-style options for a profile |

Any variable name that does not match a known pattern produces a parse error.


## Profiles

A profile variable defines a set of additional exclusion patterns.

```
tree_docs = [
    "tests/",
    ".github",
    "benches"
]
```

**Rules:**

- The variable name must start with `tree` (e.g. `tree_docs`, `tree_apis`).

- Each quoted string in the list is an exclusion pattern.

- Profile names are case insensitive and normalized to lowercase internally.

- When a profile is active, its exclusion patterns are applied **in addition to** global exclusions.

See [profiles.md](profiles.md) for detailed profile behavior.


## Tags

Tag variables define CLI-style options

**Global tags:**

```
tags = [
    "--level=3",
    "--dirsfirst"
]
```

**Profile tags:**

```
tree_docs_tags = [
    "--level=2"
]
```

**Rules:**

- Global tags apply to all tree outputs unless overridden

- Profile tags apply only when the corresponding profile is active

- Profile tags merge with global tags and override conflicting values

- The profile tag variable name must follow the pattern `<profile_name>_tags`

See [tags.md](tags.md) for detailed behavior

> **Note:** Tags are parsed in the current MVP but not yet applied


## Parse Errors

The parser produces structured errors for invalid syntax, including:

- Unclosed brackets

- Missing quotes around list elements

- Unknown variable names (not matching any recognized pattern)

- Malformed variable assignments

When a parse error occurs, `tree-it` prints a clear error message and exits with a non-zero status code.


## Complete Example
 
```
# ==========================
# .treeignore
# ==========================
 
# Global exclusions — apply to every tree
node_modules/
dist/
build/
/target
.git/
*.log
**/*.tmp
 
# Profile: documentation view
tree_docs = [
    "tests/",
    ".github/",
    "benches/",
    "scripts/"
]
 
# Profile: public-facing view
tree_public = [
    "internal/",
    ".env",
    "*.secret"
]
 
# Global tags (parsed, not yet applied)
tags = [
    "--level=3"
]
 
# Profile-specific tags (parsed, not yet applied)
tree_docs_tags = [
    "--level=2"
]
```