# Supported-Platform — Core Metadata field (historical)

**Defined in:** PEP 314 (Core Metadata 1.1, 2003)
**Current spec:** Core Metadata 2.6
**Status:** Exists but semantics never specified

## What the spec says

> "Binary distributions containing a PKG-INFO file will use the
> Supported-Platform field in their metadata to specify the OS and
> CPU for which the binary distribution was compiled. The semantics
> of the Supported-Platform field are not specified in this PEP."

That sentence has survived every Core Metadata revision from 1.1
through 2.6 — over 20 years — unchanged.

## Historical examples of use

```
Supported-Platform: RedHat 7.2
Supported-Platform: i386-win32-2791
```

The field was designed for OS + CPU identification of binary
distributions, not for Python implementation identity.

## Why we do NOT propose repurposing this field

1. Its documented scope is OS and CPU, not Python implementation
2. Its semantics are undefined, making any new use potentially
   conflicting with existing (undocumented) uses
3. The name is misleading for an interpreter requirement
4. The plural field has no defined logical model (OR? AND?)
5. It applies to "binary distributions" — not to sdists

## The correct approach

Requires-Implementation fills a distinct semantic role:

| Field | Scope | Semantics |
|---|---|---|
| Supported-Platform | OS / CPU for binary dists | Undefined |
| Requires-Python | Python version | Normative installer constraint |
| Requires-Implementation | Python implementation | Proposed normative constraint |

When reviewers ask "why not use Supported-Platform?", cite this file.
