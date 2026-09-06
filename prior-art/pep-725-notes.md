# PEP 725 — Specifying external dependencies in pyproject.toml

**URL:** https://peps.python.org/pep-0725/
**Status:** Draft (active discussion as of September 2026)
**Authors:** Ralf Gommers, Pradyun Gedam, Jaime Rodriguez-Guerra
**Relevance:** High — Paul Moore redirected this proposal to PEP 725

## What it does

Defines a standard way to declare external (non-PyPI) dependencies
in pyproject.toml using a DepURL format:

```toml
[external]
build-requires = ["dep:generic/cmake", "dep:virtual/compiler/c"]
host-requires = ["dep:generic/openssl"]
dependencies = ["dep:generic/libpq"]
```

## Core Metadata mapping

| Field | Core Metadata |
|---|---|
| `build-requires` | N/A |
| `host-requires` | N/A |
| `dependencies` | Requires-External-Dep |

**Critical finding:** build-requires and host-requires do NOT appear
in Core Metadata. Only runtime dependencies become Requires-External-Dep.

## Virtual dependency examples in current PEP 725

```
dep:virtual/compiler/c
dep:virtual/compiler/cpp
dep:virtual/compiler/rust
dep:virtual/interface/blas
dep:virtual/interface/lapack
```

Python implementations are NOT currently defined as virtual dependencies.

## Why PEP 725 does not currently solve the sdist problem

1. build-requires / host-requires have Core Metadata: N/A
2. No interpreter virtual namespace is defined
3. The Python interpreter is not modelled as an external dependency
4. Even if it were, build-time requirements cannot currently trigger
   pre-build candidate rejection via Core Metadata

## What would need to change in PEP 725

For PEP 725 to solve the sdist implementation compatibility problem:

- A virtual interpreter namespace would need to be defined
  (e.g. dep:virtual/interpreter/cpython)
- Semantics for "the running interpreter satisfies this requirement"
  would need to be specified
- The requirement would need to be exposed in Core Metadata or the
  Simple API in a form installers can act on before attempting a build

## Paul Moore's suggestion (September 6, 2026)

Paul suggested taking the sdist build-failure case to PEP 725:

> "it sounds like something that would be better handled using PEP 725,
> in particular the external.build-requires and/or external.host-requires
> fields in pyproject.toml, combined with a DepURL which can identify
> the Python implementation available in the build/host environment."

## Ralf Gommers's response (September 6, 2026)

Ralf (PEP 725 co-author) said he does not understand the link to PEP 725
for a CPython-only package, suggesting the two proposals address
different problems.

## Open question for PEP 725 thread

Could the Python interpreter itself be modelled as a virtual build
dependency, and could that requirement be made visible to a frontend
before the sdist build is attempted?
