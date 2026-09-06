# PEP 425 / Platform Compatibility Tags

## Purpose

PEP 425 introduced compatibility tags for built distributions, especially
wheels.

The current platform compatibility tag specification is the authoritative
reference for modern wheel tag behavior.

## Relevant model

A wheel's compatibility is represented using tags containing dimensions such
as:

```text
python tag
abi tag
platform tag
```

The Python tag identifies the implementation and Python version supported by
the built artifact.

Examples include implementation-specific tags such as:

```text
cp311
pp311
```

## Important distinction

Wheel tags describe the **built distribution artifact**.

They do not form a general release-level Core Metadata declaration.

This distinction matters because an sdist has not yet been built for a
specific target environment.

A typical situation is:

```text
Project release
├── source distribution
├── CPython Linux wheel
├── CPython Windows wheel
└── CPython macOS wheel
```

Each wheel can describe its own compatibility through its tags.

The sdist does not acquire an equivalent implementation tag merely because
one or more compatible wheels exist.

## Why this matters to the research

The proposal being investigated is not intended to replace wheel tags.

Instead, it asks whether a release-level implementation declaration could
provide information before a compatible wheel is available.

## Current conclusion

Wheel tags solve the artifact-level problem.

They do not directly answer the research question:

> How should a source distribution declare release-level Python
> implementation compatibility before it is built?

That is the boundary between this prior art and the proposed research.
