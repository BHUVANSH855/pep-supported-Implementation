# Design Decision Matrix

This document records the current design hypothesis. It is not a proposed
standard.

## Candidate field

Current working name:

```text
Supported-Implementation
```

Proposed `pyproject.toml` spelling:

```toml
supported-implementation = ["cpython", "pypy"]
```

The final name is unresolved.

## Semantic model

The current hypothesis is a **positive support declaration**.

For example:

```toml
supported-implementation = ["cpython", "pypy"]
```

means that the project explicitly supports those Python implementations.

It does not necessarily mean that every implementation not listed is
technically incapable of running the software.

This distinction is important for avoiding stale negative compatibility
claims.

## Value vocabulary

Values would use the implementation identity represented by:

```python
sys.implementation.name
```

This follows the implementation identity model established by PEP 421.

The field should not create a second incompatible naming system.

## Version constraints

Implementation versions are currently out of scope.

For example, this research does not currently propose:

```toml
supported-implementation = [
    "cpython >= 3.12"
]
```

Python version compatibility already has:

```toml
requires-python = ">=3.12"
```

Combining implementation identity and implementation version constraints
requires additional design work.

## ABI features

ABI characteristics are also out of scope.

Examples include:

- free-threaded vs GIL-enabled CPython;
- debug builds;
- pointer width;
- other ABI-level characteristics.

PEP 780 is relevant prior art for these dimensions.

A future implementation-support mechanism should compose with ABI feature
metadata rather than duplicate it.

## Absence of the field

Current preferred interpretation:

> If the field is absent, the project makes no standardized implementation
> compatibility claim.

This avoids treating older packages as incompatible merely because they were
published before the field existed.

The exact installer behavior remains unresolved.

## Empty list

The meaning of:

```toml
supported-implementation = []
```

is unresolved.

Possible interpretations include:

- invalid metadata;
- no supported implementations;
- no declaration.

The preferred direction is to prohibit ambiguous empty declarations rather than
give an empty list surprising semantics.

This requires a final specification decision.

## Multiple values

Multiple implementation values use OR semantics.

For example:

```toml
supported-implementation = ["cpython", "pypy"]
```

means:

```text
CPython OR PyPy
```

It does not mean that the project requires both implementations.

## Runtime vs build-time

The field is currently being considered primarily for release/runtime
compatibility.

Build-time implementation requirements are a separate question.

For example:

```text
This package runs only on CPython
```

is different from:

```text
The sdist must be built using CPython
```

The second may overlap with PEP 725 and should not automatically be encoded
using the same field.

## Installer behavior

Current research hypothesis:

- missing field: no implementation compatibility decision;
- field present and implementation listed: compatible according to the
  declaration;
- field present and implementation absent: tooling may warn or avoid the
  candidate, but hard rejection is not currently assumed.

Whether this should be:

- informational;
- warning-producing;
- candidate-filtering;
- or mandatory rejection

remains open.

## Wheel interaction

Wheel tags remain authoritative for wheel artifact compatibility.

The proposed field would describe the release/project rather than replacing
wheel tags.

For example, a project could declare:

```toml
supported-implementation = ["cpython"]
```

while publishing several CPython-specific wheels with different platform
and ABI tags.

The metadata declaration does not replace those artifact-level tags.

## Sdist interaction

This is the strongest motivation for investigating the field.

A source distribution can contain static Core Metadata under PEP 643.

If implementation support were represented in Core Metadata, a metadata
consumer could potentially inspect that declaration without executing the
package's build process.

The practical benefit depends on the metadata being available to the
consumer. PEP 658 and PEP 714 provide mechanisms for repositories to serve
Core Metadata separately, but this metadata serving is not universally
guaranteed.

## Core Metadata location

Core Metadata is a plausible location because it already describes
distribution-level facts and is carried by both wheels and conforming source
distributions.

PEP 621 would also need to define the corresponding `[project]` field if the
metadata became a standardized `pyproject.toml` project field.

This would require a subsequent standards change rather than an arbitrary new
key in `[project]`.

## Decision summary

| Question | Current position | Confidence |
|---|---|---|
| Is implementation identity already represented elsewhere? | Yes | High |
| Do wheel tags solve built-wheel compatibility? | Yes | High |
| Do dependency markers solve conditional dependencies? | Yes | High |
| Do classifiers provide implementation information? | Yes | High |
| Is there a dedicated normative release-level implementation field? | No | High |
| Is there evidence of implementation-specific packages? | Yes | High |
| Is an sdist pre-build compatibility gap demonstrated? | Yes, in specific cases | Medium/High |
| Is a new Core Metadata field definitely required? | Not established | Low |
| Is `Supported-Implementation` the final name? | No | Low |
| Should unsupported implementations be rejected? | Unresolved | Low |
| Should build-time implementation requirements use the same field? | Unresolved | Low |
| Are ABI features covered by implementation identity? | No | High |
