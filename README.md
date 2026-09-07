# Research: Python Implementation Support Metadata

Research and prior art for a possible Python packaging standard for declaring
**release-level Python implementation support**.

> **Current status (2026-09-07): research / pre-PEP.**
>
> This repository does **not** claim that a new Core Metadata field is
> necessary. It documents a narrower empirical question and the evidence
> collected so far.

## The question

The original idea was a `Requires-Implementation` Core Metadata field.

The research question is now deliberately broader and more neutral:

> **Does Python packaging need a normative release-level declaration of
> supported Python implementations, distinct from descriptive implementation
> classifiers and artifact compatibility metadata?**

If the answer is yes, a second question follows:

> **Should that declaration live in Core Metadata, and should installers treat
> it as a candidate-selection constraint?**

This distinction matters. The repository is investigating the problem before
committing to a field name or design.

---

## What is already solved?

Python packaging already has several separate compatibility mechanisms.

| Question | Existing mechanism | Status |
|---|---|---|
| Which Python versions? | `Requires-Python` | standardized, installer-relevant |
| Which dependencies apply in this environment? | PEP 508 markers | standardized |
| Which interpreter implementation does the environment have? | `sys.implementation`, `implementation_name`, `platform_python_implementation` | standardized |
| Which ABI features does the environment have? | PEP 780 proposal | active draft |
| Can this built wheel run here? | PEP 425 / wheel tags | standardized |
| Which wheel variant is appropriate? | PEP 825 proposal | active draft |
| What external build/host dependencies exist? | PEP 725 proposal | active draft |
| Which implementations does a project say it supports? | Trove classifiers | descriptive |
| Which implementations does a **release** normatively support? | — | **unresolved research question** |

The proposal therefore should not be framed as “Python has no implementation
metadata”. It does.

The question is whether there is a missing **semantic layer**.

---

## The key distinction

Consider a release that publishes:

```text
example-1.0-py3-none-any.whl
example-1.0.tar.gz
```

The wheel tag says the artifact does not require implementation-specific
features.

The producer may nevertheless say:

```text
CPython only
```

Those statements can both be true.

That gives us four separate concepts:

```text
Requirement
    What the software must have.

Compatibility
    What an artifact can run on.

Support
    What the producer is willing to support.

Evidence
    What CI, wheels, source checks, or runtime tests demonstrate.
```

The research is specifically about the **support** layer.

---

## Strongest empirical cases found

The highest-value cases are releases where the producer explicitly says
“CPython only” while the published wheel uses an implementation-generic
`py3` tag.

### RestrictedPython 8.5

PyPI currently lists:

```text
Requires-Python: >=3.10,<3.16
restrictedpython-8.5.tar.gz
restrictedpython-8.5-py3-none-any.whl
Programming Language :: Python :: Implementation :: CPython
```

The project also explicitly states that RestrictedPython only supports CPython
and not PyPy or other implementations.

This is the cleanest mature example because the artifact is
`py3-none-any`, while the release support statement is implementation-specific.

Source: https://pypi.org/project/RestrictedPython/8.5/

### HAX 0.3.0

PyPI publishes:

```text
hax-0.3.0.tar.gz
hax-0.3.0-py3-none-any.whl
```

and describes HAX as supporting CPython 3.7+.

The source contains an explicit runtime check:

```python
if implementation.name != "cpython":
    raise RuntimeError("HAX only supports CPython!")
```

This is unusually strong evidence because the restriction is not merely
documentary: the program enforces it.

Sources:
- https://pypi.org/project/hax/
- https://github.com/brandtbucher/hax/blob/add83a96a13458d66c42a8f58860e8fc25520fe4/hax/_checks.py

### Likepy 0.3.0

PyPI lists:

```text
Requires-Python: >=3.6,<3.12
likepy-0.3.0.tar.gz
likepy-0.3.0-py3-none-any.whl
Programming Language :: Python :: Implementation :: CPython
```

and the description explicitly says it only supports CPython.

Source: https://pypi.org/project/likepy/

### simple-ctx-log 0.0.3

A newer, independent example:

```text
simple_ctx_log-0.0.3.tar.gz
simple_ctx_log-0.0.3-py3-none-any.whl
```

Its project description says it uses `sys._getframe` and identifies that
feature as CPython-only.

Source: https://pypi.org/project/simple-ctx-log/

### TribeCore 4.7.3

A current 2026 example:

```text
tribecore-4.7.3.tar.gz
tribecore-4.7.3-py3-none-<platform>.whl
```

The project explicitly states:

```text
CPython only.
PyPy and other Python implementations are not supported.
```

The project explains that `py3-none-{platform}` is intentionally used so the
same wheel works across Python 3.x versions on a given platform.

Source: https://pypi.org/project/tribecore/

This is important because it shows the phenomenon is not restricted to
`py3-none-any`; an implementation-generic Python tag can coexist with a
producer-level CPython-only support policy even when the wheel is
platform-specific.

---

## What these examples do and do not prove

They demonstrate a real semantic mismatch:

```text
artifact compatibility
        ≠
producer support policy
```

They do **not** prove that:

- every CPython-only project needs a new field;
- installers should reject every package whose classifier omits PyPy;
- classifiers are inadequate for their existing descriptive purpose;
- wheel tags are inadequate for built-artifact compatibility;
- PEP 725 cannot be extended;
- PEP 825 could not evolve;
- Core Metadata is necessarily the correct place for the new information.

Those remain design questions.

---

## The strongest counter-evidence

The repository deliberately records cases that argue against overreach.

### Implementation-specific code is not the same as implementation restriction

Projects can inspect `sys.implementation` or use implementation-specific
build paths while still supporting multiple implementations.

Examples include projects such as psutil and wakepy.

### CPython-only dependencies do not imply a CPython-only release

Autobahn is a useful control case: the project supports multiple Python
implementations while some optional/native components have narrower
implementation support.

Therefore the proposed metadata must be **producer-declared**, not inferred
from dependencies or source-code checks.

### Native packages are not automatically evidence for a new release field

For packages whose wheels are already tagged `cp...`, wheel metadata may already
give an installer enough information to avoid an incompatible wheel.

Those packages remain useful for studying source-build and ABI boundaries, but
they are weaker evidence for a release-level gap.

---

## Current architecture

The evidence currently supports this layered model:

```text
RELEASE / PROJECT SUPPORT
    "Which implementations does the producer support?"
    ← candidate research gap

ENVIRONMENT
    "Which implementation / ABI does this interpreter provide?"
    ← sys.implementation / PEP 508 / PEP 780

ARTIFACT
    "Can this wheel run here?"
    ← PEP 425 / wheel tags / PEP 825

BUILD
    "What is needed to build this source?"
    ← PEP 517 / PEP 725 / PEP 804

USER POLICY
    "Should source builds be attempted?"
    ← installer policy such as --only-binary
```

The important word is **layered**. A future field should not replace the
existing mechanisms.

---

## Candidate solutions under investigation

1. Keep implementation support purely descriptive via classifiers.
2. Give existing classifiers stronger semantics.
3. Add a new positive support declaration, e.g.
   `Supported-Implementation`.
4. Add a requirement-like field, e.g.
   `Requires-Implementation`.
5. Extend index-level metadata rather than Core Metadata.
6. Use wheel variants for artifact compatibility and solve only source/release
   support elsewhere.
7. Add a source-build policy mechanism such as a future “do not automatically
   build this sdist” signal.
8. Do nothing and improve tooling around existing signals.

The repository does not currently select one of these.

---

## Important prior art

- **PEP 421** — defines `sys.implementation`.
- **PEP 425** — built-distribution compatibility tags.
- **PEP 508 / dependency specifiers** — implementation environment markers.
- **PEP 621** — project metadata declaration in `pyproject.toml`.
- **PEP 625** — sdist filename format.
- **PEP 643** — static metadata consistency for source distributions.
- **PEP 658 / PEP 714** — serving Core Metadata through the Simple API.
- **PEP 725** — external build/host/runtime dependency metadata.
- **PEP 780** — ABI feature environment markers.
- **PEP 794** — release-level Core Metadata precedent.
- **PEP 817 / PEP 825** — wheel variant compatibility.
- **Trove implementation classifiers** — existing descriptive vocabulary.
- **sdist-build avoidance discussions** — adjacent source-build policy problem.

See `prior-art/` for the evidence and the exact boundary of each mechanism.

---

## Evidence standard

This repository intentionally distinguishes:

### Direct evidence

Examples:

- published PyPI metadata;
- published filenames;
- project documentation;
- source code containing an explicit compatibility check;
- normative packaging specifications;
- actual standards discussions.

### Interpretation

A reasoned mapping from the evidence to the research question.

### Hypothesis

A proposed mechanism that still needs ecosystem validation.

The repository should never turn an inference such as:

```text
no PyPy wheel
```

into:

```text
PyPy unsupported
```

Likewise:

```text
sys.implementation.name is inspected
```

must not automatically become:

```text
package is CPython-only
```

---

## What would make the proposal fail?

The proposal should probably be abandoned or substantially changed if we
find that:

1. classifiers can safely acquire the required semantics without compatibility
   problems;
2. existing wheel/index metadata already provides an equivalent release-level
   answer;
3. a source-build policy mechanism solves the real user problem without
   needing implementation support metadata;
4. the number of meaningful residual cases is too small;
5. downstream consumers do not need the information;
6. the field would be too ambiguous to use safely.

That is intentional. The goal is to determine whether a standard is warranted,
not to justify one.

---

## Next step

The broad package search is now sufficient for a first research corpus.

The next work should be **consolidation and adversarial validation**:

1. freeze the strongest residual cases;
2. verify each against existing mechanisms;
3. classify false positives;
4. compare Core Metadata against index-level metadata;
5. identify actual consumers;
6. update the Python.org discussion with evidence, not advocacy.

See:

- `evidence/residual-cases.md`
- `evidence/methodology.md`
- `design/decision-matrix.md`
- `design/open-questions.md`
- `discussion.md`
