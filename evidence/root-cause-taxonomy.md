# Root-Cause Taxonomy for Implementation Restrictions

## Purpose

A statement such as **"CPython only"** is an observation, not yet a standardization argument.

Implementation-specific behavior can arise from materially different causes. This taxonomy is part of the evidence methodology so that the repository does not count every implementation-related case as evidence for new release metadata.

The central rule is:

> **Observed restriction ≠ root cause ≠ metadata gap ≠ standardization requirement.**

A second rule is equally important:

> **A technical incompatibility does not automatically imply that the correct packaging mechanism is Core Metadata.**

Every candidate residual case should therefore be classified before it is counted as support for a new metadata mechanism.

---

## Taxonomy

### A — Runtime semantic restriction

The release fundamentally depends on behavior that is specific to one Python implementation and does not provide an equivalent supported path elsewhere.

Examples might include:

```text
CPython-specific runtime semantics
CPython-only runtime guard
implementation-specific behavior with no fallback
```

A particularly strong example is an explicit runtime guard such as:

```python
if implementation.name != "cpython":
    raise RuntimeError("...")
```

This is potentially the strongest category for release-level implementation-support metadata.

It still requires evidence that:

1. the restriction applies to the release as a whole or to the relevant installation functionality;
2. the behavior is not merely a temporary implementation defect;
3. no supported fallback exists;
4. existing artifact/build metadata cannot adequately represent the restriction;
5. the restriction creates a concrete consumer decision.

Category A therefore identifies a **technical cause**, not automatically a **metadata residual**.

---

### B — Build/toolchain restriction

The release can conceptually run on another implementation, but the source cannot currently be built there.

Examples include:

```text
build backend requires CPython
compiler/toolchain integration only works under CPython
extension build process assumes CPython
implementation-specific build helper
```

This is primarily a **build-environment** question.

It should be tested against:

* build requirements;
* host requirements;
* external dependencies;
* compiler/toolchain requirements;
* wheel artifact compatibility;
* PEP 725;
* other build-system mechanisms.

A build restriction must not automatically become:

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

A package may be portable after being built elsewhere, or the build limitation may belong to the build environment rather than the resulting distribution.

---

### C — Alternative-implementation bug or missing functionality

The package itself may be intended to support another implementation, but that implementation currently lacks a feature or has a bug.

Conceptually:

```text
package
    ↓
requires feature X
    ↓
alternative implementation lacks/fails X
```

Examples include:

* an alternative interpreter lacking a CPython API;
* a compatibility-layer defect;
* an interpreter-specific bug;
* a feature that has not yet been implemented.

This is not necessarily a package-level compatibility declaration.

The correct long-term fix may be in the alternative implementation rather than package metadata.

Such cases should not be counted as strong residual evidence unless there remains a stable producer-level support boundary that packaging metadata must express.

A temporary implementation bug is especially weak evidence for a permanent exclusion field.

---

### D — ABI / configuration restriction

The apparent implementation restriction is actually an ABI or interpreter-configuration dimension.

Examples include:

```text
free-threading
debug builds
32/64-bit differences
specific CPython ABI
platform-specific native ABI
interpreter configuration
```

These should first be evaluated against:

* wheel ABI tags;
* platform tags;
* Python tags;
* PEP 780;
* PEP 825;
* other artifact compatibility mechanisms.

This category is deliberately excluded from the core residual set unless the existing ABI/configuration machinery demonstrably cannot represent the relevant release-level fact.

A native extension is therefore not automatically evidence for an implementation-support field.

---

### E — Private implementation usage

The producer intentionally depends on implementation internals or private APIs.

Examples include:

```text
CPython private API
CPython object internals
implementation-specific private behavior
unsupported low-level interpreter assumptions
```

This is a difficult policy boundary.

The fact that software is intentionally written for one implementation does not automatically establish a need for standardized installer rejection metadata.

For example:

```text
private CPython API
```

may explain why a maintainer chooses not to support PyPy, but the packaging consequence depends on whether the release is actually incompatible, whether a fallback exists, and what consumer action is required.

Cases in this category are important because they expose the difference between:

```text
technical dependency on an implementation
```

and:

```text
normative package-selection requirement
```

They should not be promoted to strong residual evidence without a separate argument about why packaging metadata should encode the support boundary.

---

### F — Explicit support-policy declaration

The maintainer explicitly states a support boundary such as:

```text
CPython supported
PyPy unsupported
```

but the available evidence does not establish a technical incompatibility sufficient to explain the boundary.

This is valuable evidence about **producer support policy**.

It is weaker evidence for resolver-level exclusion.

This category is especially important for testing a possible positive support declaration such as:

```text
Supported-Implementation: cpython
```

because it represents the semantic proposition:

```text
producer chooses to support X
```

rather than necessarily:

```text
X is technically required
```

A policy declaration should therefore not automatically become a hard installation constraint.

---

### G — Conditional or fallback implementation support

The package behaves differently by implementation but provides a supported fallback.

Examples:

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

Implementation-specific source code does not imply implementation-specific release support.

A package may legitimately contain:

```text
if implementation == X:
    use optimized path
else:
    use portable path
```

while supporting both environments.

Category G should therefore generally be treated as **counter-evidence or a design constraint**, not residual evidence.

---

### H — Dependency/component restriction

A dependency or optional component may support only one implementation while the parent release supports several.

Therefore:

```text
CPython-only dependency
        ≠
CPython-only parent release
```

Support cannot safely be inferred transitively without inspecting:

* dependency markers;
* optional extras;
* fallback behavior;
* conditional imports;
* feature boundaries;
* runtime behavior.

For example:

```text
package A
    |
    +--> dependency B
            |
            +--> CPython-specific component
```

does not establish:

```text
A = CPython-only
```

because A may use B only conditionally or provide another implementation path.

---

## I — Artifact-level implementation restriction

An additional category is useful when the implementation restriction belongs directly to a published artifact.

Examples include:

```text
pp3-none-any
cp311-none-any
cp311-cp311-...
```

The wheel compatibility-tag system already provides an implementation-level mechanism for these cases.

Historical `packaging` Issue #311 is important prior art here: it demonstrated that pure-Python artifacts can use implementation-specific tags such as:

```text
pp3-none-any
```

when the artifact itself is intended for a particular implementation.

This category should therefore normally be classified as:

```text
existing artifact mechanism
```

rather than as evidence for a new release-level support field.

The important distinction is:

```text
artifact requires implementation X
```

versus:

```text
artifact is generic,
but producer declares release support only for X
```

Only the latter remains potentially relevant to the current metadata question.

---

## J — Security or guarantee boundary

Some projects restrict implementations because their correctness, isolation, security, or formal guarantees depend on interpreter behavior.

For example:

```text
implementation difference
        ↓
security guarantee cannot be maintained
        ↓
producer refuses support
```

This category overlaps with F and sometimes A/E.

It is useful to record separately because security-sensitive support policies may require stronger semantics than ordinary portability claims.

A security-support statement should not automatically be treated as an ordinary resolver exclusion.

The research must determine whether the producer is asserting:

```text
technically cannot operate
```

or:

```text
cannot provide the project's stated security guarantee
```

Those are materially different claims.

---

## Multiple classifications

A release may legitimately belong to more than one category.

For example:

```text
private CPython API
+
runtime guard
+
explicit support policy
```

may be classified as:

```text
A + E + F
```

The classifications describe different dimensions of the same case.

They should not be forced into a single mutually exclusive bucket when doing so would lose important information.

The research should nevertheless identify a **primary cause** when possible.

---

## Cause versus evidence strength

The taxonomy is not itself a confidence ranking.

For example:

```text
A — runtime restriction
```

can be weakly evidenced if it is inferred from documentation alone.

Conversely:

```text
F — support policy
```

can be strongly evidenced if the producer explicitly documents the policy for the exact release.

Therefore record separately:

```text
root cause
+
evidence strength
+
residual status
```

Do not treat category A as automatically stronger than category F.

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
   Verify published evidence
          │
          ▼
    What is the actual cause?
          │
    ┌─────┼─────────┬────────┬────────┐
    ▼     ▼         ▼        ▼        ▼
 runtime build      ABI    private   policy/
 semantic toolchain       API       docs
    │     │         │        │        │
    A     B         D        E        F
          │
          ▼
    C when an alternative
    implementation defect
    is the actual cause
          │
          ▼
    G when fallback/
    conditional behavior exists
          │
          ▼
    H when dependency/component
    scope is the actual issue
          │
          ▼
    I when artifact compatibility
    already represents the restriction
          │
          ▼
    J when a security/guarantee
    boundary is material
          │
          ▼
    Check existing mechanisms
          │
     ┌────┼──────────────┐
     ▼    ▼              ▼
represented partly       not represented
     │    covered             │
     ▼    ▼                   ▼
 control investigate      residual candidate
 case      gap                 │
                               ▼
                    consumer decision?
                               │
                               ▼
                    incremental benefit?
```

Then separately ask:

1. Is the release-level support fact explicit?
2. Is the exact release identified?
3. Is `Requires-Python` insufficient?
4. Are PEP 508 mechanisms insufficient?
5. Are wheel Python/ABI/platform tags sufficient for every published artifact?
6. Is the issue actually ABI/configuration?
7. Is it actually a build/host requirement covered by PEP 725?
8. Is it an alternative-implementation defect?
9. Is private implementation usage the actual reason?
10. Is there a fallback or conditional path?
11. Could an existing classifier communicate the positive support fact?
12. Does the restriction apply to the entire release or only a feature/component?
13. What would a consumer do differently?
14. Must that decision happen before installation/build?
15. Would standardized metadata provide an incremental benefit?

Only after these questions are answered should a case be counted as a strong residual case.

---

## Residual status decision

The taxonomy should lead to one of the following outcomes:

```text
Existing mechanism sufficient
        ↓
control/non-residual

Cause understood
+
existing mechanism insufficient
+
consumer decision established
        ↓
residual candidate

Residual candidate
+
incremental consumer benefit demonstrated
+
alternative solutions considered
        ↓
strong residual

Strong residual corpus
+
generalizable semantic class
+
acceptable producer/consumer tradeoff
        ↓
potential standardization case
```

The final step remains independent of the root-cause category.

---

## Current corpus classification

The present evidence should be treated as follows:

| Case                 | Current classification                                    | Research role                                                     |
| -------------------- | --------------------------------------------------------- | ----------------------------------------------------------------- |
| HAX 0.3.0            | A + F candidate                                           | strongest runtime-enforced case                                   |
| simple-ctx-log 0.0.3 | E + F candidate                                           | private-API/support-boundary case                                 |
| RestrictedPython 8.5 | F + J, with implementation-specific technical assumptions | security/support-policy case                                      |
| Likepy 0.3.0         | F pending deeper source/root-cause evidence               | classifier/support-policy test                                    |
| TribeCore 4.7.3      | F/control                                                 | implementation-generic native-artifact boundary                   |
| Guppy3               | D + E/control                                             | ABI/configuration boundary                                        |
| Specialist           | A/F candidate pending deeper validation                   | implementation + version boundary                                 |
| PyInstaller          | candidate; A/B/D/E/F require release-specific audit       | high-value implementation-rejection case                          |
| PageBloomFilter      | B/D adjacent case                                         | native build/ABI boundary                                         |
| coverage.py          | G/control + historical transition evidence                | fallback and support-transition case                              |
| Autobahn             | H/G control                                               | dependency/fallback control                                       |
| psutil               | G/control                                                 | implementation-specific behavior without automatic narrow support |

These classifications are provisional.

They are intended to make uncertainty visible rather than hide it.

In particular, **candidate** does not mean **confirmed residual**.

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
exact release identified
        +
explicit release-level support boundary
        +
root cause classified
        +
whole-release or relevant-installation scope established
        +
not merely an alternative-interpreter bug
        +
not merely ABI/configuration
        +
not merely build/host metadata
        +
not merely a private-API observation
        +
not merely a support-policy statement
        +
not already represented by artifact metadata
        +
Requires-Python insufficient
        +
PEP 508 insufficient
        +
actual consumer decision identified
        +
decision occurs before relevant cost/failure
        +
incremental consumer benefit demonstrated
        +
credible alternative mechanisms considered
```

This higher threshold is necessary to make the research useful to people who disagree with the proposal.

---

## Examples of classification mistakes to avoid

### Mistake 1 — Generic wheel means universal support

Incorrect:

```text
py3-none-any
    ↓
all implementations supported
```

Correct:

```text
py3-none-any
    ↓
artifact has no implementation-specific Python tag
```

Producer support must be established separately.

### Mistake 2 — CPython classifier means hard exclusion

Incorrect:

```text
CPython classifier
    ↓
reject PyPy
```

Correct:

```text
CPython classifier
    ↓
descriptive implementation classification
```

Its suitability as a normative support declaration is a separate research question.

### Mistake 3 — Runtime failure proves resolver metadata is required

Incorrect:

```text
PyPy installation fails
    ↓
new metadata required
```

Correct:

```text
PyPy installation fails
    ↓
identify root cause
    ↓
check existing mechanisms
    ↓
identify consumer decision
    ↓
evaluate incremental metadata benefit
```

### Mistake 4 — CPython-only dependency makes parent CPython-only

Incorrect:

```text
dependency B = CPython-only
    ↓
package A = CPython-only
```

Correct:

```text
dependency B = CPython-only
    ↓
inspect A's dependency conditions/fallbacks
    ↓
determine A's effective support
```

### Mistake 5 — Native extension requires new metadata

Incorrect:

```text
native extension
    ↓
implementation metadata required
```

Correct:

```text
native extension
    ↓
inspect ABI/platform/Python tags
    ↓
inspect build requirements
    ↓
determine whether a release-level implementation fact remains
```

---

## Why this taxonomy matters

The goal is not to maximize the number of examples.

The goal is to determine whether there is a **residual semantic class** that:

1. occurs in real releases;
2. matters before installation/build or another clearly identified consumer action;
3. cannot safely be represented by existing mechanisms;
4. has a useful consumer action;
5. is stable enough to standardize;
6. can be maintained reliably by publishers;
7. is worth standardizing despite publisher and ecosystem adoption costs.

The taxonomy therefore acts as a filter between:

```text
implementation-related observation
```

and:

```text
evidence for a new metadata standard
```

If the corpus does not produce a residual class satisfying these conditions, the correct research conclusion is:

**no new standard.**
