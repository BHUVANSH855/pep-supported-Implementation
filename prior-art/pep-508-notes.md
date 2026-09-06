# PEP 508 — Dependency specification for Python Software Packages

**URL:** https://peps.python.org/pep-0508/
**Status:** Final
**Relevance:** Medium — defines environment markers including
implementation_name, but at the dependency level not the distribution level

## Relevant environment markers

| Marker | Corresponds to |
|---|---|
| `implementation_name` | `sys.implementation.name` |
| `implementation_version` | `sys.implementation.version` |
| `platform_python_implementation` | `platform.python_implementation()` |

## What markers solve

Conditional dependency installation. Example:

```
Requires-Dist: cffi; implementation_name == "pypy"
```

Meaning: install cffi only when running on PyPy.

## Critical semantic distinction

A marker on `Requires-Dist` describes when a dependency is needed.
It does NOT describe whether the package itself is compatible with
a given implementation.

There is no syntactic way to express "this package only runs on
CPython" using Requires-Dist markers. You would have no dependency
to conditionally include.

## What PEP 508 does NOT solve

Distribution-level implementation compatibility. The gap this
research addresses is precisely NOT covered by PEP 508.

## Key distinction for discussions

When reviewers suggest "just use environment markers":

> Markers describe dependency conditions.
> Requires-Implementation would describe the distribution itself.
> These are different semantic layers.
