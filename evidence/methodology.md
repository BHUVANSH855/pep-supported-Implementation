# Evidence Methodology

## Purpose

This repository is intended to be useful to packaging maintainers and
standards authors, not merely persuasive.

The methodology therefore separates:

1. observed facts;
2. root-cause classification;
3. interpretation;
4. proposed semantics.

A package enters the strong evidence set only when the relevant facts can be
independently checked.

The core methodological rule is:

> **Observed restriction ≠ root cause ≠ standardization requirement.**

---

## Evidence hierarchy

### Level 1 — normative specifications

Highest-confidence sources:

* Python Packaging User Guide specifications;
* accepted/final PEPs;
* current draft PEPs when explicitly identified as drafts.

Relevant material includes:

* Core Metadata;
* PEP 425;
* PEP 508 / dependency specifiers;
* PEP 643;
* PEP 658 / PEP 714;
* PEP 725;
* PEP 780;
* PEP 825.

### Level 2 — published distribution metadata

Examples:

* PyPI release metadata;
* `Requires-Python`;
* classifiers;
* distribution filenames;
* presence/absence of sdist and wheels.

### Level 3 — producer documentation

Examples:

* README;
* installation documentation;
* support matrix;
* release notes.

### Level 4 — source evidence

Examples:

```python
if sys.implementation.name != "cpython":
    ...
```

This is particularly useful when it directly enforces the claimed restriction.

### Level 5 — ecosystem discussion

Discourse discussions are evidence of design arguments and ecosystem practice,
not normative specifications.

---

## Root-cause classification

Every claimed implementation restriction should first be classified using
`evidence/root-cause-taxonomy.md`.

At minimum, distinguish:

* runtime semantic restriction;
* build-toolchain restriction;
* alternative-implementation bug/workaround;
* ABI/configuration restriction;
* private implementation usage;
* documentation/support policy only;
* conditional/fallback support;
* dependency/component restriction.

Do not count all of these as equivalent evidence.

In particular:

```text
build only on CPython
```

must not automatically become:

```text
runtime supports only CPython
```

and:

```text
CPython-specific code
```

must not automatically become:

```text
other implementations are unsupported
```

---

## What counts as a strong residual case?

A strong case should answer these questions:

| Question                                                            | Required?                 |
| ------------------------------------------------------------------- | ------------------------- |
| Does the producer explicitly state implementation support?          | Yes                       |
| Is the statement attached to a concrete release/version?            | Yes                       |
| Is the Python version range separately expressible?                 | Yes                       |
| Is the wheel implementation tag generic or otherwise insufficient?  | Strongly preferred        |
| Is an sdist available?                                              | Strongly preferred        |
| Can PEP 508 express the package's own restriction?                  | Must be analyzed          |
| Is this actually an ABI issue?                                      | Must be checked           |
| Is this actually a build/host requirement?                          | Must be checked           |
| Could PEP 725 or another build mechanism solve it?                  | Must be checked           |
| Is this actually an alternative-implementation bug?                 | Must be checked           |
| Is private implementation usage the real reason?                    | Must be checked           |
| Could an existing classifier communicate the positive support fact? | Must be tested            |
| Is there a false-positive/fallback explanation?                     | Must be checked           |
| Would metadata change a pre-install/pre-build decision?             | Yes for a strong residual |
| Is there a concrete consumer benefit?                               | Yes for a strong residual |

The standard is deliberately higher than:

```text
CPython only + py3-none-any
```

---

## What we explicitly do NOT infer

### No PyPy wheel

Does **not** imply:

```text
PyPy unsupported
```

A project may simply choose not to publish a PyPy wheel while supporting
source builds.

### CPython classifier

Does not prove:

```text
all other implementations are technically incompatible
```

It proves, at most, that the project has declared CPython in its descriptive
classification.

A major research question is whether classifiers can safely provide positive
support information without changing their existing meaning.

### `sys.implementation` check

Does not automatically mean the project is CPython-only.

The code may use a CPython-specific optimization and provide a fallback.

### Native extension

Does not automatically justify a new field.

Wheel tags and ABI mechanisms may already solve the artifact-selection problem.

### CPython-only dependency

Does not automatically make the parent package CPython-only.

The parent may provide an alternative implementation path.

---

## Residual-case test

For each candidate release, evaluate:

```text
                   ┌── Requires-Python
                   │
                   ├── PEP 508 markers
                   │
                   ├── wheel Python/ABI/platform tags
Release candidate ─┼── PEP 780 ABI features
                   │
                   ├── PEP 825 variants
                   │
                   ├── PEP 725 build/host metadata
                   │
                   ├── source-build policy
                   │
                   └── classifiers
                            │
                            ▼
                  Root cause classified?
                            │
                            ▼
                 Is the producer's support
                 claim machine-actionable?
                            │
                    ┌───────┴───────┐
                    │               │
                   YES              NO
                    │               │
              existing path     residual candidate
```

The answer must be based on the actual release, not a hypothetical package.

---

## Classifier experiment

The classifier question is now a first-class research track.

Test at least these cases.

### A — Positive support

```text
CPython classifier
+
documentation/CI says CPython supported
```

Can the classifier safely mean:

> The publisher explicitly supports CPython for this release.

### B — Missing implementation classifier

```text
CPython classifier
no PyPy classifier
```

Does absence mean:

```text
unsupported
```

or:

```text
unknown / not declared
```

The repository currently assumes the safer interpretation:

```text
absence = no normative declaration
```

### C — Classifier disagreement

Find releases where:

```text
classifier says CPython
documentation/CI says PyPy supported
```

or the reverse.

These are high-value cases because they directly test whether classifiers
can be made machine-actionable without changing historical metadata semantics.

### D — No classifier

Find releases with explicit implementation-support statements but no
implementation classifier.

This measures classifier coverage.

### E — Support transitions

Track releases where implementation support changes over time.

This tests whether positive support declarations can remain accurate and
maintainable.

---

## Consumer-benefit test

A metadata proposal should not be justified only by:

```text
"the information would be nice to know."
```

For each strong case ask:

```text
current resolver path
        ↓
candidate selected
        ↓
sdist build attempted
        ↓
failure / wasted work
```

versus:

```text
support metadata available
        ↓
candidate rejected or deprioritized
        ↓
alternative candidate selected
```

Then estimate:

* build time avoided;
* network/download work avoided;
* repeated failure avoided;
* diagnostic improvement;
* resolver quality improvement.

Build-failure caching is a serious competing solution and must be compared
directly.

The relevant question is:

> **What does standardized metadata provide that cached build outcomes cannot
> provide?**

---

## False-positive controls

The corpus deliberately includes projects that:

* support CPython and PyPy;
* inspect `sys.implementation`;
* conditionally build native extensions;
* have CPython-only optional dependencies;
* publish fewer wheels than they test;
* have platform-specific but implementation-generic wheels;
* use CPython-specific code while retaining a fallback.

These controls are essential because otherwise an automated scan would
overestimate the problem.

---

## Quantitative claims

The current repository does **not** claim a PyPI-wide prevalence percentage.

The current corpus is intentionally enriched for implementation-related
cases.

Therefore a statement such as:

```text
6/32 packages are CPython-only
```

must not be presented as:

```text
18.75% of PyPI is CPython-only
```

A real prevalence estimate would require a reproducible corpus and a defined
sampling methodology, ideally using PyPI distribution metadata or a public
dataset.

---

## Evidence ledger

Every strong case should eventually have:

```text
project
version
release date
Requires-Python
classifiers
sdist
wheel filenames
implementation statement
root-cause classification
source/runtime evidence
dependency markers
ABI evidence
build/host evidence
candidate-selection effect
consumer benefit
alternative solution analysis
source URLs
confidence
```

This makes the repository auditable by someone who disagrees with the
proposal.

---

## Reproducibility target

A future automated study should ideally use a release-level dataset such as
PyPI distribution metadata and classify:

```text
release
├── Requires-Python
├── classifiers
├── Requires-Dist
├── filenames
├── packagetype
└── upload time
```

Then enrich a deliberately selected subset with:

```text
README/docs
source checks
CI matrices
build tests
implementation behavior
```

The current repository has not performed a statistically representative
PyPI-wide analysis, so it should not claim one.
