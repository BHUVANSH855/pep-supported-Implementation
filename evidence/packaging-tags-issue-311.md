# `packaging` Issue #311 — Historical Pure-Python Implementation Tags

## Purpose

This is historical prior art for implementation-specific wheel tags on pure-Python distributions.

The issue was opened in 2020 around `packaging.tags` handling of a PyPy-specific pure-Python wheel tag, `pp3-none-any`.

The motivating case was the PEP 615 backport, which had both C and pure-Python implementations and wanted to publish a pure-Python wheel specifically for PyPy while allowing other environments to fall back to the sdist. The issue reported that newer pip/packaging behavior no longer considered the `pp3-none-any` wheel compatible, whereas older pip behavior did.

The issue was subsequently resolved, and `packaging` 21.3 explicitly records:

```text
Add a `pp3-none-any` tag
```

for issue #311.

---

## What the issue demonstrates

The important historical fact is:

```text
pure Python artifact
        +
implementation-specific compatibility
        ↓
implementation-specific wheel tag
```

For example:

```text
pp3-none-any
```

can describe a pure-Python wheel intended specifically for PyPy 3.

The current compatibility-tag specification likewise defines the Python tag as identifying the implementation and version required by a distribution, with `pp` representing PyPy and `py` representing generic Python compatibility.

Therefore:

```text
pure Python
```

does **not** necessarily mean:

```text
all Python implementations
```

---

## Why this matters to the current research

Issue #311 is important because it establishes a strong historical boundary:

> Wheel compatibility tags are capable of representing implementation-specific restrictions even when the distribution contains only Python code.

That means the current research must **not** argue:

```text
pure-Python implementation restriction
        ↓
wheel tags cannot express it
        ↓
new metadata required
```

That argument would be incorrect.

Instead, the relevant question is narrower:

```text
Can the artifact itself honestly be represented as:

    py3-none-any

while the producer nevertheless wants to communicate:

    this release is not supported on PyPy
```

That is a materially different problem.

---

## Artifact compatibility versus producer support

Issue #311 helps establish the distinction between two concepts.

### Artifact-level compatibility

A wheel tag answers a question about the artifact:

```text
Can this particular distribution artifact be used
with this implementation/environment?
```

For an implementation-specific pure-Python wheel:

```text
pp3-none-any
```

the artifact itself is explicitly restricted to PyPy 3.

### Producer-declared release support

The current research is investigating a different possibility:

```text
artifact:
    py3-none-any

producer:
    supports CPython
    does not support PyPy
```

Here the producer may consider the generic wheel technically appropriate as an artifact while still defining a narrower support policy for the release.

The semantic distinction under investigation is therefore:

```text
artifact compatibility
        ≠
producer-declared implementation support
```

Issue #311 provides historical evidence for the first side of that distinction; it does not establish the second.

---

## Why implementation-specific tags remain the correct mechanism for artifact restrictions

The current packaging specification states that compatibility tags allow build tools to mark distributions as compatible with specific platforms and allow installers to understand which distributions are compatible with the running system. The Python tag identifies the implementation and version required by the distribution.

Consequently, when the **artifact itself** requires a particular implementation, the existing tag mechanism should remain the primary mechanism.

Examples include:

```text
pp3-none-any
cp311-none-any
cp311-cp311-manylinux_2_17_x86_64
```

The existence of a producer-level implementation-support question must not be used as a reason to weaken or replace wheel compatibility tags.

---

## The important residual distinction

The current research is interested in cases closer to:

```text
release R
    |
    +-- wheel: py3-none-any
    |
    +-- sdist
    |
    +-- Requires-Python: version constraint only
    |
    +-- producer: CPython only
    |
    +-- runtime/source evidence: implementation restriction
```

The question is then:

> If `py3-none-any` is an honest artifact-level compatibility description, where should the producer's narrower release-level support boundary be represented?

This is the unresolved question.

It is **not**:

> How can wheel tags express an implementation-specific artifact?

Issue #311 already demonstrates that they can.

---

## Important distinction: generic artifact versus generic support

A `py3-none-any` wheel means the artifact is not declaring an implementation-specific Python tag.

It does not, by itself, establish that the producer intends to support every Python implementation.

Likewise:

```text
pp3-none-any
```

does not mean that the entire project is universally PyPy-only.

It describes the compatibility of that particular artifact.

Therefore the research should avoid interpreting either direction too strongly:

```text
generic wheel
    ≠
universal producer support
```

and:

```text
implementation-specific wheel
    ≠
project-wide implementation policy
```

---

## Historical use case: implementation-specific fallback

Issue #311 is particularly useful because the motivating use case involved multiple implementation paths.

Conceptually:

```text
CPython
    ↓
C implementation / CPython-specific artifact

PyPy
    ↓
pure-Python implementation
    ↓
pp3-none-any

other environments
    ↓
sdist / other compatible path
```

This illustrates why artifact compatibility and release support must be analyzed separately.

A package may legitimately publish different artifacts for different implementations without requiring a new release-level support declaration.

---

## What Issue #311 does NOT prove

Issue #311 does **not** prove that:

* wheel tags are inadequate for implementation restrictions;
* pure-Python packages need a new implementation metadata field;
* `py3-none-any` is semantically equivalent to universal implementation support;
* installers should infer producer support from the absence of implementation-specific tags;
* implementation-specific wheel tags should be replaced;
* `Requires-Implementation` is necessary;
* `Supported-Implementation` is necessary;
* producer support policy should be encoded in wheel filenames;
* every CPython-only package should publish a CPython-specific wheel.

In particular:

```text
no `pp` wheel
        ≠
PyPy unsupported
```

A producer may publish only a generic wheel and support PyPy, or publish a generic wheel while intentionally supporting only a subset of implementations.

The artifact alone cannot establish the latter support policy.

---

## Relationship to the strongest residual cases

The historical tag evidence should be applied to current cases before classifying them as residual.

For a package claiming CPython-only support, ask:

```text
1. Does the artifact actually require CPython?
        ↓
2. If yes, can the wheel tag express that?
        ↓
3. If yes, is the current artifact incorrectly generic?
        ↓
4. If no, why can the artifact remain generic?
        ↓
5. Is the remaining restriction a release-level producer support boundary?
```

Only cases that survive this sequence remain candidates for a separate support mechanism.

For example:

```text
HAX
    runtime CPython guard
    +
    py3-none-any
```

is more interesting than a package that simply publishes:

```text
cp311-...
```

because the latter already has an artifact-level compatibility mechanism.

Likewise, a package whose CPython restriction is actually caused by a native extension should first be examined through ABI and wheel-tag semantics.

---

## Research consequence

Issue #311 strengthens the existing methodology rather than weakening it.

It establishes:

```text
implementation-specific artifact compatibility
        ↓
already has a packaging mechanism
```

Therefore a new metadata proposal must establish something narrower:

```text
release-level producer support
        ↓
not adequately represented by artifact tags
        ↓
not adequately represented by Requires-Python
        ↓
not adequately represented by dependency markers
        ↓
not adequately represented by build/host metadata
        ↓
creates a concrete consumer decision
```

This is the standard required before treating an implementation-support field as a genuine residual.

---

## Current classification

The issue should therefore be classified as:

```text
Historical prior art
+
Existing-mechanism evidence
+
Boundary/control case
```

It is **not** evidence that wheel tags are inadequate.

If anything, it provides evidence for the opposite conclusion:

> When an artifact itself is implementation-specific, the wheel compatibility-tag system already has a mechanism for expressing that restriction, including for pure-Python artifacts.

The unresolved research question lies beyond that boundary:

> What machine-readable mechanism, if any, should describe a producer's release-level implementation support when the artifact itself can honestly remain implementation-generic?

---

## Current conclusion

Keep wheel tags as the artifact-level compatibility mechanism.

Do not use `packaging` Issue #311 as an argument to replace or extend wheel tags merely because implementation-specific compatibility exists.

Instead, use it as a control case demonstrating that:

```text
implementation-specific artifact restriction
        ↓
existing wheel-tag mechanism
```

while continuing to investigate:

```text
implementation-generic artifact
        +
narrower producer support policy
        ↓
?
```

That remaining question is the relevant potential metadata gap.

No conclusion should yet be drawn that the answer must be a new Core Metadata field.
