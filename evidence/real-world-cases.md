# Real-World Cases

This document records concrete examples relevant to implementation-level
compatibility.

## 1. Guppy3

Guppy3 is the strongest current example.

Its build configuration explicitly checks:

```python
sys.implementation.name != "cpython"
```

This is direct evidence that the build process itself contains an
implementation-specific compatibility condition.

The project's published information also states that:

- CPython is supported;
- PyPy is unsupported;
- other Python implementations are unsupported;
- free-threaded CPython is unsupported.

### Why this matters

The package demonstrates two different compatibility dimensions:

```text
Python implementation identity
        +
ABI/configuration characteristics
```

It therefore supports investigating an implementation-level metadata field,
but also demonstrates why implementation identity cannot represent all
compatibility information.

### What it does not prove

Guppy3 does not prove that:

- every CPython-only project needs metadata;
- installers must reject PyPy;
- a new Core Metadata field is required;
- PEP 725 cannot solve the build-time part of the problem.

It is evidence of a concrete compatibility boundary.

## 2. Tooling at scale

Discussion around this proposal identified non-installer use cases such as:

- testing large package sets against multiple implementations;
- fuzzing packages against different interpreters;
- pre-filtering packages before expensive builds;
- compatibility testing across CPython, PyPy, and other implementations.

These use cases are relevant because the consumer of compatibility metadata does
not necessarily have to be an installer.

A research question is therefore:

> Is implementation compatibility information useful enough to justify a
> standard machine-readable declaration even if installers only use it
> conservatively?

## 3. Packages that adapt to implementations

Not every implementation difference is a hard compatibility boundary.

Projects such as:

- `aiohttp`;
- `multidict`;
- `coverage.py`;
- `python-zstandard`;

provide examples of software that can adapt to different environments,
provide fallbacks, or conditionally use implementation-specific features.

This is important counter-evidence.

The existence of CPython-specific projects does not mean that implementation
metadata should be mandatory for ordinary packages.

## 4. Trove classifiers

Projects can already use classifiers such as:

```text
Programming Language :: Python :: Implementation :: CPython
```

and:

```text
Programming Language :: Python :: Implementation :: PyPy
```

This demonstrates that the ecosystem already has a vocabulary for describing
implementation targeting.

The limitation is semantic rather than syntactic:

classifiers are descriptive classification metadata, not a defined
candidate-selection compatibility constraint.

## 5. Sdist build avoidance

Packaging discussions have identified cases where an installer may select an
sdist and attempt a build that is likely to fail or is not intended for
ordinary installation.

This is broader than Python implementation compatibility.

The research uses these discussions as evidence that:

> pre-build knowledge about whether an sdist is an appropriate installation
> candidate can have practical value.

They do not establish that implementation metadata is the correct solution.

## 6. Pure-Python detection discussion

A separate packaging discussion asked how tooling could determine whether an
sdist is pure Python without attempting a complete build.

This is another example of the broader problem:

```text
source distribution
        ↓
unknown build characteristics
        ↓
tool must perform work to discover them
```

The existence of this problem reinforces the need to distinguish:

- implementation compatibility;
- build characteristics;
- artifact compatibility.

A single metadata field should not be overloaded to cover all three.

## 7. Research interpretation

The real-world evidence currently supports:

- implementation-specific compatibility is real;
- implementation-specific build logic is real;
- pre-build metadata can be useful;
- implementation information already exists descriptively;
- wheel artifacts already have strong implementation compatibility metadata;
- the unresolved problem is primarily the release/sdist layer.

It does not yet support a claim that one particular metadata design is
necessary.
