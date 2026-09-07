# Research Status — 2026-09-07

## Current conclusion

The research has demonstrated a real semantic distinction between:

```text
artifact compatibility
```

and:

```text
producer-declared implementation support
```

The strongest evidence comes from releases that publish implementation-generic
Python wheel tags while explicitly declaring CPython-only support.

However, the research has now reached an important refinement:

> A statement such as "CPython only" is not by itself a residual metadata case.

The cause must first be classified. Some restrictions may be build-toolchain
limitations, alternative-implementation bugs, ABI/configuration constraints,
private implementation usage, dependency restrictions, or documentation-only
support policy.

The repository now treats this root-cause distinction as a required part of
the evidence standard.

---

## Current strongest research question

The question is no longer:

> Do we need `Requires-Implementation`?

The working question is:

> **Are there real release-level implementation support constraints for which
> existing classifiers, artifact tags, build metadata, and cached build
> outcomes are insufficient, and where standardized pre-build metadata would
> materially improve package selection?**

A second question follows:

> **If such cases exist, can existing classifiers safely provide the required
> positive support semantics, or is a new standardized metadata mechanism
> justified?**

---

## What is established

### High confidence

* Python version compatibility and Python implementation identity are distinct
  dimensions.
* Wheel artifact compatibility and producer release support are distinct
  dimensions.
* `Requires-Python` does not express implementation identity.
* PEP 508 markers provide implementation-aware dependency conditions but do
  not directly declare that the distribution itself is invalid on another
  implementation.
* Real releases can combine implementation-generic wheel tags with explicit
  CPython-only support statements.
* Missing wheels and implementation-specific source code are not sufficient
  evidence of unsupported implementations.

### Medium confidence / still under investigation

* Existing implementation classifiers may provide useful positive support
  information.
* A positive `Supported-Implementation` concept may be semantically safer than
  `Requires-Implementation`.
* Standardized metadata may avoid expensive source-build attempts in some
  real-world cases.
* Build-result caching is useful but may not be equivalent to pre-install
  support metadata.
* PEP 725 may solve some cases that initially look like implementation
  compatibility problems.

### Not established

The research has **not** established that a new Core Metadata field is
necessary.

It has also not established:

* that classifiers are insufficient;
* that every CPython-only package should be represented in metadata;
* that implementation support should cause hard resolver rejection;
* that Core Metadata is necessarily the correct layer;
* that the current corpus represents PyPI-wide prevalence.

---

## Current evidence posture

The current corpus should be read by root cause:

| Case                 | Provisional role                                                                 |
| -------------------- | -------------------------------------------------------------------------------- |
| RestrictedPython 8.5 | strongest artifact/support mismatch; root cause still needs final classification |
| HAX 0.3.0            | strong runtime-enforced implementation restriction                               |
| Likepy 0.3.0         | support-policy/classifier case pending deeper technical evidence                 |
| simple-ctx-log 0.0.3 | recent implementation-specific case; source validation pending                   |
| TribeCore 4.7.3      | important artifact mismatch, but native/build analysis required                  |
| winuvloop 0.2.5      | dependency-graph/control case                                                    |
| psutil               | false-positive control                                                           |
| Autobahn             | dependency/fallback control                                                      |
| Guppy3               | ABI/artifact boundary control                                                    |
| cffi                 | classifier-semantics investigation                                               |

This is intentionally more conservative than simply counting all five
CPython-only examples as proof of a missing standard.

---

## Current hypothesis about field design

If a new mechanism is eventually justified, a positive producer declaration such
as:

```text
Supported-Implementation: cpython
```

appears semantically safer than:

```text
Requires-Implementation: cpython
```

because the observed use cases are primarily support declarations.

The current hypothesis is:

```text
presence
    = positive support assertion

absence
    = no normative support assertion
```

not:

```text
absence
    = unsupported
```

and not yet:

```text
presence
    = mandatory resolver exclusion of all unlisted implementations
```

This is a research hypothesis, not a recommendation.

---

## Main unresolved questions

1. Can classifiers safely be given positive support semantics?
2. What does omission mean?
3. Can a classifier remain advisory while a new field becomes normative?
4. Which root-cause classes actually deserve packaging metadata?
5. Can PEP 725 represent the build-only subset?
6. What does a resolver do differently before building an sdist?
7. What does metadata provide that failed-build caching cannot?
8. Is Core Metadata or index-level metadata the better layer?
9. How much publisher effort would a new field impose?
10. Can implementation support change safely across releases?
11. What is the minimum vocabulary for implementation identity?
12. How should ABI/configuration dimensions interact with implementation
    support?

---

## Next research phase

### Phase 1 — case audit

Audit every current candidate release with the root-cause taxonomy.

### Phase 2 — classifier study

Build a release-level set containing:

* positive implementation classifier;
* no implementation classifier;
* classifier/documentation disagreement;
* classifier/CI disagreement;
* support transitions.

### Phase 3 — resolver simulation

For each strong residual case compare:

```text
current behavior
classifier-based behavior
new-metadata behavior
```

and record the actual candidate-selection consequence.

### Phase 4 — alternative solutions

Explicitly test:

* classifiers;
* PEP 725;
* source-build policy;
* failed-build caching;
* index metadata;
* wheel variants;
* no new standard.

### Phase 5 — decision gate

Only draft a PEP if the evidence shows all of:

```text
real residual cases
+
root cause appropriate for package support metadata
+
existing mechanisms insufficient
+
measurable consumer benefit
+
credible publisher adoption
+
clear semantics
```

Otherwise the defensible result is:

```text
no new standard
```

---

## Accountability rule

This repository should make it easy for a packaging maintainer to disagree with
the conclusion.

Every strong claim should therefore identify:

```text
what was observed
what was inferred
what remains uncertain
what alternative explanation was tested
```

The purpose of the research is not to win the standards discussion. It is to
determine whether the proposed semantic layer is actually necessary.
