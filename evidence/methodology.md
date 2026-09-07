# Evidence Methodology

## Purpose

This repository is intended to be useful to packaging maintainers and
standards authors, not merely persuasive.

The methodology therefore separates:

1. observed facts;
2. interpretation;
3. proposed semantics.

A package enters the strong evidence set only when the relevant facts can be
independently checked.

---

## Evidence hierarchy

### Level 1 — normative specifications

Highest-confidence sources:

- Python Packaging User Guide specifications;
- accepted/final PEPs;
- current draft PEPs when explicitly identified as drafts.

Examples:

- Core Metadata
- PEP 425
- PEP 508 / dependency specifiers
- PEP 643
- PEP 658 / PEP 714
- PEP 780
- PEP 825

### Level 2 — published distribution metadata

Examples:

- PyPI release metadata;
- `Requires-Python`;
- classifiers;
- distribution filenames;
- presence/absence of sdist and wheels.

### Level 3 — producer documentation

Examples:

- README;
- installation documentation;
- support matrix;
- release notes.

### Level 4 — source evidence

Examples:

```python
if sys.implementation.name != "cpython":
    ...
```

This is particularly useful when it directly enforces the claimed
restriction.

### Level 5 — ecosystem discussion

Discourse discussions are evidence of design arguments and ecosystem
practice, not normative specifications.

---

## What counts as a strong residual case?

A strong case should answer these questions:

| Question | Required? |
|---|---|
| Does the producer explicitly state implementation support? | Yes |
| Is the statement attached to a concrete release/version? | Preferably |
| Is the Python version range separately expressible? | Yes |
| Is the wheel implementation tag generic or otherwise insufficient? | Strongly preferred |
| Is an sdist available? | Strongly preferred |
| Can PEP 508 express the package's own restriction? | Must be analyzed |
| Is this actually an ABI issue? | Must be checked |
| Could a source-build policy signal solve the operational problem? | Must be checked |
| Is there a false-positive explanation? | Must be checked |

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

### `sys.implementation` check

Does not automatically mean the project is CPython-only.

The code may use a CPython-specific optimization and provide a fallback.

### Native extension

Does not automatically justify a new field.

Wheel tags may already solve the artifact-selection problem.

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
                   ├── build/external metadata
                   │
                   └── classifiers
                            │
                            ▼
                 Is the producer's support
                 claim machine-actionable?
                            │
                    ┌───────┴───────┐
                    │               │
                   YES              NO
                    │               │
              existing path     residual case
```

The answer must be based on the actual release, not a hypothetical package.

---

## False-positive controls

The corpus deliberately includes projects that:

- support CPython and PyPy;
- inspect `sys.implementation`;
- conditionally build native extensions;
- have CPython-only optional dependencies;
- publish fewer wheels than they test;
- have platform-specific but implementation-generic wheels.

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
source/runtime evidence
dependency markers
ABI evidence
candidate-selection effect
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
```

The current repository has not performed a statistically representative
PyPI-wide analysis, so it should not claim one.
