# pep-supported-implementation

Background research for a potential `Supported-Implementation` field
in Python Core Metadata.

---

## The question

Python packaging already standardises:

- Implementation identity at runtime — `sys.implementation.name` (PEP 421)
- Implementation-compatible wheel selection — wheel filename tags (PEP 425)
- Implementation-specific dependencies — `implementation_name` markers (PEP 508)
- Python version constraints — `Requires-Python` in Core Metadata

What does not exist is a distribution-level Core Metadata field
expressing which Python implementations a project is known to support.

This gap is most visible for **source distributions**. A wheel encodes
its implementation in the filename; pip filters on that. An sdist has no
equivalent signal. When a package only works on CPython and ships only an
sdist, an installer on PyPy or GraalPy has no way to know that before
attempting a build.

Paul Moore (CPython core developer, packaging PEP delegate) confirmed:

> "Currently, no I don't think there is [a standard way for an installer
> to know a package is CPython-only from sdist metadata before building]."

---

## Proposed direction

A new optional Core Metadata field — provisionally `Supported-Implementation`
— declaring which Python implementations a release is known to work on:

```toml
[project]
requires-python = ">=3.12"
supported-implementation = ["cpython", "pypy"]
```

Values correspond to `sys.implementation.name` (PEP 421). The set is
open — no central registry needed. A missing field means no claim has
been made; existing packages are unaffected.

The framing is **positive** ("known to work on") rather than exclusionary
("incompatible with"). Absence does not mean incompatible — it means the
maintainer has not made a declaration. This avoids stale metadata
becoming a hard block as alternate implementations improve.

---

## Status

Pre-PEP community discussion. No PEP number assigned.

Discourse thread:
https://discuss.python.org/t/pre-pep-requires-implementation-declaring-python-implementation-compatibility-in-core-metadata/108898

---

## Repository structure

```
prior-art/      — relevant PEPs and specifications
evidence/       — real-world packages and tooling that demonstrate the gap
design/         — open design questions
discussion.md   — summary of the Discourse discussion so far
```

---

## Author

Bhuvansh (BHUVANSH855 on discuss.python.org)
CPython contributor, Python docs translation coordinator (Punjabi),
builder of PyRift (CPython/PyPy comparison toolkit)