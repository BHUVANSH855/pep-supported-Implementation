# Requires-Implementation Research

Research and prior art for a possible Python packaging mechanism to declare
Python implementation compatibility at the distribution/release level.

> **Status: research only.**
>
> This repository is not a PEP draft and does not currently argue that a new
> `Requires-Implementation` field is necessary or that it is the correct final
> design.

## Research question

Python packaging already has several mechanisms for describing different
dimensions of compatibility:

- `Requires-Python` describes supported Python versions.
- Wheel tags describe the compatibility of a built wheel.
- PEP 508 environment markers can condition dependencies on the Python
  implementation.
- Trove classifiers can describe the implementation a project targets.
- Core Metadata can be stored statically in source distributions.

The open question is narrower:

> Is there a standard, machine-readable way for a distribution release to
> declare Python implementation compatibility before an sdist is built?

For example, suppose a release supports CPython but not PyPy. If the only
available artifact is an sdist, an installer or external compatibility tool
may need to start the build process before discovering that fact.

This repository investigates whether that is a real ecosystem gap, what
existing mechanisms already cover it, and whether a new metadata mechanism
would improve the situation.

## What is already solved?

| Problem | Existing mechanism | Assessment |
|---|---|---|
| Python version compatibility | `Requires-Python` | Standard |
| Wheel implementation compatibility | Wheel/platform tags | Standard |
| Conditional dependencies | PEP 508 markers | Standard |
| Descriptive implementation classification | Trove classifiers | Exists, but descriptive |
| Static metadata in sdists | PEP 643 | Standard |
| Serving metadata without downloading an artifact | PEP 658 / PEP 714 | Supported by the repository API |
| Release-level implementation compatibility for sdists | No dedicated field | Open question |
| sdist build requiring a specific Python implementation | No dedicated mechanism | Open question |

The important distinction is that these mechanisms describe different things.
A wheel tag describes a **built artifact**. A dependency marker describes
whether a **dependency applies**. A Trove classifier describes how a project
is **classified**. None of those is currently a normative release-level
compatibility declaration for Python implementation identity.

## Concrete evidence

The strongest concrete case currently collected is `guppy3`.

Its build configuration explicitly checks:

```python
sys.implementation.name != "cpython"
```

and its project documentation states that PyPy and other Python
implementations are unsupported.

This demonstrates a real implementation-specific compatibility boundary.
It does not, by itself, prove that every such project needs new metadata.

The repository also records ecosystem discussions about avoiding unnecessary
sdist builds and tooling use cases where compatibility information could be
useful before building thousands of packages.

## Proposed direction under investigation

The current working design uses:

```text
Supported-Implementation
```

with a TOML spelling such as:

```toml
supported-implementation = ["cpython", "pypy"]
```

The values would use the implementation identity represented by
`sys.implementation.name`.

This is intentionally a **positive support declaration** rather than a
negative requirement such as:

```text
Requires-Implementation: cpython
```

The reason is staleness.

A declaration that says:

```text
Requires-Implementation: cpython
```

can become incorrect if another implementation later becomes compatible.

A declaration that says:

```text
Supported-Implementation: cpython
```

can instead be interpreted as a statement about the implementations the
maintainer has declared supported for that release.

This is only a research hypothesis at this stage.

## Important scope boundary: ABI compatibility

Implementation identity is not the same thing as complete interpreter
compatibility.

For example, a project may support CPython generally while not supporting
a particular ABI configuration such as free-threaded CPython.

PEP 780 is therefore important prior art. It investigates ABI features as
environment markers, including characteristics such as free-threading and
debug builds.

This research therefore treats implementation identity and ABI features as
separate compatibility dimensions.

A future design should not attempt to use a single
`Supported-Implementation` field to encode every interpreter compatibility
property.

## Important scope boundary: build vs runtime

There are at least two different questions:

1. Does the released software support this Python implementation at runtime?
2. Does building the software require this Python implementation?

These should not automatically be treated as the same metadata field.

The first question is a natural candidate for a release-level support
declaration.

The second may overlap with the dependency/build-environment work being
discussed in PEP 725.

This repository therefore keeps build-time implementation requirements as a
separate open design question.

## What this repository does not claim

This research does **not** currently claim that:

- every CPython-only package needs new metadata;
- wheel tags are insufficient for wheels;
- Trove classifiers have no value;
- PEP 725 cannot be extended to address some of this problem;
- implementation identity is enough to describe interpreter compatibility;
- an installer should necessarily reject an unsupported implementation;
- `Requires-Implementation` is definitely the correct field name;
- the proposed field should necessarily be normative.

The purpose of the repository is to establish the problem and evaluate the
design space before making those claims.

## Research structure

### Prior art

The `prior-art/` directory documents relevant packaging standards and
discussions:

- PEP 421 — `sys.implementation`
- PEP 425 / platform compatibility tags
- PEP 508 — environment markers
- PEP 621 — project metadata in `pyproject.toml`
- PEP 625 — source distribution filenames
- PEP 643 — static metadata in sdists
- PEP 658 / PEP 714 — serving Core Metadata through the Simple API
- PEP 725 — external/build/host dependencies
- PEP 780 — ABI features
- PEP 794 — import names and namespaces
- Trove classifiers
- `Requires-Python`
- sdist build-avoidance discussions

### Evidence

The `evidence/` directory records real-world examples, tooling discussions,
and historical packaging issues.

### Design

The `design/` directory records the current design hypothesis and unresolved
questions.

## Current conclusion

There is enough evidence to justify continued investigation of
release-level Python implementation compatibility, especially for sdists.

There is **not yet enough evidence to conclude that a new Core Metadata field
is the only or best solution**.

The next useful step is to compare:

1. a new Core Metadata field;
2. an extension of existing metadata mechanisms;
3. an extension of PEP 725;
4. improved use of wheel artifacts/tags;
5. a purely informational declaration;
6. an installer-facing normative compatibility field.

The research should converge on a precise problem statement before proposing
a standards change.
