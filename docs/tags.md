# Tags

**Project Name:** .treeignore  
**Project Manager:** @ciroalo  
**Last Revision Date:** April 3, 2026  
**Version:** 0.1  
**Status:** Stable (MVP subset)

This document describes the tag system in `.treeignore`


## Overview

Tags are CLI-style defined inside `.treeignore` that influence how trees are generated 
or displayed. They allow you to embed default options in the configuration file rather 
than passing them on the command line every time.

> **Note:** In the current MVP, tags are parsed and validated during configuration loading,
but they are not yet applied to tree generation. Full tag behavior is planned for future versions.


## Global Tags

Global tags apply to all tree outputs unless overridden by profile tags or CLI flags.

```
tags = [
    "--level=3",
    "--dirsfirst"
]
```

The variable must be named exactly `tags`


## Profile Tags

Profile tags apply only when the corresponding profile is active. They are defined using a variable named `<profile_name>_tags`:

```
tree_docs_tags = [
    "--level=2",
    "--ascii"
]
```

When a profile is selected, its tags are merged with global tags:

- Tags present only in global apply as-is

- Tags present only int he profile apply as-is

- Tags present in both, the profile value overrided the global value


## Precedence

When determining the effective set of options for a tree, the following priority applies:

```
CLI flags > profile tags > global tags
```

This means:

1. A CLI flag always wins over any tag defined in `.treeignore`

2. A profile tag overrides a global tag for the same option

3. A global tag applies only if neither a CLI flag nor a profile tag sets that option


## Syntax

Tags follow the same variable assignment syntax as profiles:

```
tags = [
    "--option=value",
    "--flag"
]
```

Each element is a double-quoted string representing a CLI-style option


## Error Handling (Planned)

In a future version, unknown or invalid tags will produce errors. The planned behavior:

- Tags that do not correspond to any recognized `tree-it` option will cause the tool to print an error and exit with a non-zero status code

- This ensures that typos or outdated tags are caught rather than silently ignored

 
## Example
 
```
# Global: all trees default to 3 levels deep
tags = [
    "--level=3"
]
 
# Docs profile: override to 2 levels
tree_docs_tags = [
    "--level=2"
]
 
# Public profile: no custom tags (inherits global)
tree_public = [
    "internal/"
]
```
 
**Effective tags per tree (planned behavior):**
 
| Tree         | `--level` value |
|--------------|-----------------|
| General      | `3` (global)    |
| tree_docs    | `2` (profile overrides global) |
| tree_public  | `3` (global, no override)      |
 