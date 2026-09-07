# `Supported-Platform` Historical Core Metadata

**Current specification:** Core Metadata 2.6
**Source:** https://packaging.python.org/en/latest/specifications/core-metadata/

The existing `Supported-Platform` field says it can describe the OS and CPU
for which a binary distribution was compiled.

Its semantics are explicitly not specified.

Example historical values include:

```text
Supported-Platform: RedHat 7.2
Supported-Platform: i386-win32-2791
```

## Why it should not simply be repurposed

1. Its documented subject is OS/CPU, not Python implementation.
2. Its semantics are undefined.
3. Its name does not identify interpreter compatibility.
4. Its historical purpose is tied to binary distributions.
5. Reusing it would create ambiguous semantics for existing consumers.

## Current conclusion

A new implementation-support concept should not silently overload this
historical field.

The repository instead investigates whether a distinct semantic layer is
needed.
