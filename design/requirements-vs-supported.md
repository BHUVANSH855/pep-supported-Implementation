# Requirements vs Supported Implementations

A central design question is whether the metadata should express a
**requirement** or a **support declaration**.

## Model A: requirement

Example:

```text
Requires-Implementation: cpython
```

Possible interpretation:

> This release requires CPython and cannot be used with other
> implementations.

### Advantages

- Direct installer-facing semantics.
- Easy to understand.
- Similar in spirit to `Requires-Python`.

### Risks

The statement may be interpreted as an exhaustive incompatibility claim.

If an implementation that was previously unsupported becomes compatible,
older releases could contain metadata that is technically too restrictive.

For example:

```text
Release 1.0:
    Requires-Implementation: cpython

Later:
    PyPy becomes fully compatible
```

The old release would still claim that CPython is required.

## Model B: support declaration

Example:

```text
Supported-Implementation: ["cpython"]
```

Possible interpretation:

> The maintainer explicitly supports CPython for this release.

The absence of PyPy does not necessarily claim that PyPy is technically
incapable of running the software.

### Advantages

- More conservative.
- Better aligned with maintainer support policy.
- Less likely to make stale negative claims.
- Can naturally represent multiple explicitly supported implementations.

### Risks

If the field is not normative, installers may not know what to do with it.

If the field is normative, the distinction between "not listed" and
"unsupported" must be specified.

## Model C: informational classifier

Use existing or extended classifiers.

Example:

```text
Programming Language :: Python :: Implementation :: CPython
```

### Advantages

- Existing mechanism.
- Already widely understood.
- No new Core Metadata semantic category.

### Risks

- Classifiers are primarily descriptive.
- Candidate selection semantics are not defined.
- A classifier does not clearly distinguish "supported" from "project
  classification".

## Comparison

| Model | Installer semantics | Staleness risk | Existing mechanism |
|---|---|---:|---|
| `Requires-Implementation` | Strong | Higher | No |
| `Supported-Implementation` | Configurable | Lower | No |
| Trove classifier | Weak/descriptive | Low | Yes |

## Current research preference

The current research prefers investigating a positive support declaration
before a negative requirement.

That is a design preference, not a conclusion.

The final choice should depend on what semantics ecosystem stakeholders
actually need.
