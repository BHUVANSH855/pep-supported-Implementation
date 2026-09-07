# Root-Cause Taxonomy for Implementation Restrictions

## Purpose

A statement such as **"CPython only" is an observation, not yet a
standardization argument**.

The recent standards discussion identified several materially different causes
for implementation-specific behavior. This taxonomy is part of the evidence
methodology so that the repository does not count every implementation-specific
case as evidence for new release metadata.

The central rule is:

> **Observed restriction ≠ root cause ≠ standardization requirement.**

Every candidate residual case should therefore be classified before it is
counted as support for a new metadata mechanism.

---

## Taxonomy

### A — Runtime semantic restriction

The release fundamentally depends on behavior that is specific to one Python
implementation and does not provide an equivalent supported path elsewhere.

Examples might include:

```text
CPython-specific runtime semantics
CPython-only runtime guard
implementation-specific behavior with no fallback
```

This is the strongest potential category for release-level support metadata.

It still requires evidence that the restriction is a property of the release,
rather than merely an implementation bug or maintainer preference.

---

### B — Build-toolchain restriction

The release can conceptually run on another implementation, but the source
cannot currently be built there.

Examples:

```text
build backend requires CPython
compiler/toolchain integration only works under CPython
extension build process assumes CPython
```

This is primarily a **build-environment** question.

It should be tested against PEP 725 and source-build mechanisms before being
treated as evidence for runtime support metadata.

A build restriction must not automatically be converted into:

```text
Supported-Implementation: cpython
```

because:

```text
build under CPython
```

does not imply:

```text
runtime requires CPython
```

---

### C — Alternative-implementation bug or missing functionality

The package itself may be intended to support another implementation, but that
implementation currently lacks a feature or has a bug.

Conceptually:

```text
package
   ↓
requires feature X
   ↓
alternative implementation lacks/fails X
```

This is not necessarily a package-level compatibility declaration.

The correct long-term fix may be in the alternative implementation rather than
in package metadata.

Such cases should not be counted as strong residual evidence unless there is
still a stable producer-level support boundary that metadata needs to express.

---

### D — ABI / configuration restriction

The apparent implementation restriction is actually an ABI or interpreter
configuration dimension.

Examples include:

```text
free-threading
debug builds
32/64-bit differences
specific CPython ABI
```

These should first be evaluated against wheel ABI tags and PEP 780.

This category is deliberately excluded from the core residual set unless the
existing ABI machinery demonstrably cannot represent the release-level fact.

---

### E — Private implementation usage

The producer intentionally depends on implementation internals or private
APIs.

Examples:

```text
CPython private API
CPython object internals
unsupported implementation-specific behavior
```

This is a difficult policy boundary.

The fact that software is intentionally written for one implementation does
not automatically establish a need for standardized installer rejection
metadata.

The repository records these cases because they are important counterexamples
to the original broad framing.

They should not be promoted to strong residual evidence without a separate
argument about why packaging metadata should encode this support policy.

---

### F — Documentation / support-policy declaration

The maintainer says:

```text
CPython supported
PyPy unsupported
```

but the available evidence does not establish a technical incompatibility.

This is valuable evidence about **producer support policy**, but it is weaker
evidence for resolver-level exclusion.

These cases are especially important for testing the proposal for a positive
support declaration such as:

```text
Supported-Implementation: cpython
```

They do not by themselves justify making the declaration a hard constraint.

---

### G — Conditional or fallback implementation support

The package behaves differently by implementation but provides a supported
fallback.

For example:

```text
CPython path
PyPy fallback
```

or:

```text
native acceleration on CPython
portable implementation elsewhere
```

These cases are evidence against simplistic inference rules.

Implementation-specific source code does not imply implementation-specific
release support.

---

### H — Dependency/component restriction

A dependency or optional component may support only one implementation while
the parent release supports several.

Therefore:

```text
CPython-only dependency
        ≠
CPython-only parent release
```

Support cannot safely be inferred transitively without inspecting the complete
dependency and fallback behavior.

---

## Classification decision procedure

For every candidate release:

```text
                 "Implementation restriction"
                           │
                           ▼
                  Identify exact release
                           │
                           ▼
                What is the actual cause?
                           │
          ┌────────┬───────┼────────┬─────────┐
          ▼        ▼       ▼        ▼         ▼
       runtime   build    ABI     private   policy/
       semantic  toolchain       API       docs-only
          │        │       │        │         │
          ▼        ▼       ▼        ▼         ▼
          A        B       D        E         F
                   │
                   ▼
                  C when
              implementation
              defect is cause
                           │
                           ▼
              Check existing mechanisms
                           │
             ┌─────────────┼─────────────┐
             ▼             ▼             ▼
       represented     partly covered   not represented
             │             │             │
          control       investigate     residual
           case            gap          candidate
```

Then separately ask:

1. Is the release-level support fact explicit?
2. Is the exact version identified?
3. Is `Requires-Python` insufficient?
4. Are PEP 508 self-restriction mechanisms insufficient?
5. Are wheel tags sufficient for every published artifact?
6. Is the issue actually ABI/configuration?
7. Is it actually a build/host requirement covered by PEP 725?
8. Is it an alternative-implementation defect?
9. Is it merely private-API policy?
10. Could the existing classifier communicate the positive support fact?
11. What would an installer do differently before building?
12. Would that change produce a measurable consumer benefit?

Only after these questions are answered should a case be counted as a strong
residual case.

---

## Current corpus classification

The repository's present evidence should be treated as follows:

| Case                 | Current classification                             | Research role                                       |
| -------------------- | -------------------------------------------------- | --------------------------------------------------- |
| RestrictedPython 8.5 | A/F candidate; root cause needs final verification | strongest artifact/support mismatch                 |
| HAX 0.3.0            | A                                                  | strong runtime-enforced case                        |
| Likepy 0.3.0         | F pending deeper source/root-cause evidence        | classifier/support-policy test                      |
| simple-ctx-log 0.0.3 | A/F candidate pending source validation            | recent implementation-specific case                 |
| TribeCore 4.7.3      | B/D/F candidate pending native/build analysis      | artifact mismatch requiring classification          |
| winuvloop 0.2.5      | H / mixed                                          | dependency-graph control                            |
| psutil               | G/control                                          | implementation-specific code without narrow support |
| Autobahn             | H/G control                                        | narrow dependency does not imply parent restriction |
| Guppy3               | D/control                                          | ABI/artifact boundary control                       |
| cffi                 | classifier-semantics investigation                 | tests positive-support classifier hypothesis        |

These classifications are provisional. They are intended to make uncertainty
visible rather than hide it.

---

## What qualifies as a strong residual case now?

The bar is intentionally higher than simply finding:

```text
CPython only
+
py3-none-any
```

A strong residual case should additionally establish:

```text
explicit release-level support boundary
        +
root cause classified
        +
not merely an alternative-interpreter bug
        +
not merely ABI/configuration
        +
not merely a private-API policy case
        +
not already represented by existing artifact/build metadata
        +
pre-build/pre-install candidate-selection consequence
        +
concrete consumer benefit
```

This higher threshold is necessary to make the research useful to people who
disagree with the proposal.

---

## Why this taxonomy matters

The goal is not to maximize the number of examples.

The goal is to determine whether there is a **residual semantic class** that:

1. occurs in real releases;
2. matters before installation/build;
3. cannot safely be represented by existing mechanisms;
4. has a useful consumer action;
5. is worth standardizing despite publisher/adoption costs.

If the corpus does not produce such a class, the correct research conclusion is
**no new standard**.
