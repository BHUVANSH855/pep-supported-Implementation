# PEP 780 Notes — ABI Features

**URL:** https://peps.python.org/pep-0780/
**Status:** Draft / active discussion
**Relevance:** High

PEP 780 proposes `sys_abi_features` as an environment marker for ABI
characteristics of the Python interpreter.

Examples include:

- free-threading;
- GIL-enabled operation;
- debug builds;
- pointer width.

## Why this matters

Implementation identity is not the complete compatibility surface.

For example:

```text
CPython
CPython + free-threading
```

have the same implementation identity but may have different package
compatibility.

## Correct layering

```text
Supported-Implementation
    implementation identity

PEP 780
    ABI/environment features

Requires-Python
    Python version

Wheel tags
    built artifact compatibility
```

## Guppy3

Guppy3 is a useful boundary case because its published support information
distinguishes CPython from free-threaded CPython.

This demonstrates why an implementation-support field must not become a
general-purpose ABI expression language.

Current conclusion:

> PEP 780 is complementary prior art and a scope boundary.
