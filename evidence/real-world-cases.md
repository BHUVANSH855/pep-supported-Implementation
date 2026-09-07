# Real-World Cases

This document records concrete examples and ecosystem observations relevant to
implementation-level compatibility.

It is intentionally broader than `residual-cases.md`.

A real-world case demonstrates that a particular behavior, restriction, workflow,
or metadata mismatch exists. It does **not** automatically demonstrate that:

* the behavior requires new metadata;
* the metadata should be normative;
* installers should act on it;
* the information belongs in Core Metadata;
* or an implementation-support field is preferable to existing mechanisms.

The research therefore distinguishes:

```text
real-world observation
        ↓
root-cause classification
        ↓
residual case
        ↓
standardization requirement
```

Only the latter stages can support a proposal for new normative metadata.

---

## 1. Guppy3

Guppy3 is an important implementation/ABI boundary case.

Its build configuration explicitly checks:

```python
sys.implementation.name != "cpython"
```

The project's published information also states that:

* CPython is supported;
* PyPy is unsupported;
* other Python implementations are unsupported;
* free-threaded CPython is unsupported.

## Why this matters

Guppy3 demonstrates that a package can have multiple compatibility dimensions:

```text
Python implementation identity
        +
ABI/configuration characteristics
        +
Python version
        +
build/artifact compatibility
```

This makes it useful for investigating where implementation identity fits
relative to existing packaging mechanisms.

It also demonstrates why an implementation-support mechanism cannot be treated
as a universal replacement for version, ABI, or artifact metadata.

## Root-cause interpretation

The current evidence places Guppy3 primarily near:

```text
D — ABI/configuration restriction
E — private/implementation-specific usage
```

with build-time implementation checks also present.

This distinction matters.

A build-time check involving `sys.implementation` is not by itself evidence
that a release-level implementation-support field is required. The underlying
restriction may already be representable through:

* wheel tags;
* ABI information;
* Python-version metadata;
* build requirements;
* or artifact selection.

## What it does not prove

Guppy3 does not prove that:

* every CPython-only project needs metadata;
* installers must reject PyPy;
* a new Core Metadata field is required;
* PEP 725 cannot solve the build-time part of the problem;
* implementation identity alone is sufficient to describe compatibility.

Guppy3 should therefore be treated as an important **boundary/control case**,
not as the cleanest residual case.

---

## 2. Explicit CPython-only releases with generic wheels

The research has identified releases where the producer states that only
CPython is supported while the published wheel remains implementation-generic.

Examples currently under investigation include:

* RestrictedPython 8.5;
* HAX 0.3.0;
* Likepy 0.3.0;
* simple-ctx-log 0.0.3;
* TribeCore 4.7.3.

Typical artifact patterns include:

```text
py3-none-any
```

or:

```text
py3-none-{platform}
```

while the producer communicates a narrower implementation-support boundary.

These examples are analyzed in detail in:

```text
evidence/residual-cases.md
```

## Why this matters

These releases expose a potentially important distinction:

```text
artifact compatibility
        ≠
producer-declared implementation support
```

A generic wheel can mean that the artifact does not require an
implementation-specific wheel tag. It does not necessarily mean that the
producer intends to support every Python implementation.

This is precisely the semantic boundary being investigated.

## Important qualification

These cases should not all be counted as proof of a missing standard.

Each must first be classified according to its root cause:

```text
A — runtime semantic restriction
B — build/toolchain restriction
C — alternative-implementation bug/workaround
D — ABI/configuration restriction
E — private implementation usage
F — explicit support-policy declaration
G — conditional/fallback support
H — dependency/component restriction
```

In particular:

```text
producer says "CPython only"
        ≠
all non-CPython environments are technically impossible
```

and:

```text
generic wheel
        ≠
universal producer support
```

The strongest current candidate is HAX 0.3.0 because the implementation
restriction is explicitly enforced at runtime. The remaining cases require
additional root-cause and consumer-benefit validation.

---

## 3. Tooling at scale

Discussion around this proposal identified non-installer use cases such as:

* testing large package sets against multiple implementations;
* fuzzing packages against different interpreters;
* pre-filtering packages before expensive builds;
* compatibility testing across CPython, PyPy, and other implementations.

These use cases matter because the consumer of implementation compatibility
information does not necessarily have to be an installer.

A testing or research system could conceptually want:

```text
package release
        ↓
supported implementations
        ↓
select test matrix
```

without attempting installation under every implementation first.

## Research question

The relevant question is:

> Is implementation compatibility information useful enough to justify a
> standard machine-readable declaration even if installers use it only
> conservatively?

This is separate from the question:

> Should an installer reject a candidate based on the declaration?

The first could potentially have value even if the second remains controversial.

## Evidence status

**Open research question.**

The existence of these use cases demonstrates potential consumer demand, but
does not establish how frequently they occur or whether existing classifiers
are sufficient.

Quantitative claims about ecosystem-wide demand require a reproducible corpus
rather than anecdotal examples.

---

## 4. Packages that adapt to implementations

Not every implementation difference is a hard compatibility boundary.

Projects such as:

* `aiohttp`;
* `multidict`;
* `coverage.py`;
* `python-zstandard`;

provide examples of software that can adapt to different environments,
provide fallbacks, or conditionally use implementation-specific features.

## Why this is important counter-evidence

A package can contain implementation-specific code without declaring an
implementation-level exclusion.

For example:

```text
implementation-specific feature
        ↓
fallback available
        ↓
multiple implementations supported
```

Therefore:

```text
implementation-specific source code
        ≠
implementation unsupported
```

and:

```text
implementation-specific dependency
        ≠
top-level package unsupported
```

## Research implication

Any proposed metadata field must avoid forcing projects with conditional or
fallback behavior into a false binary model such as:

```text
CPython = yes
PyPy = no
```

where the actual support relationship is more nuanced.

This is one reason the semantics of a possible positive
`Supported-Implementation` field require careful investigation.

## Evidence status

**Counter-evidence / design constraint.**

These cases demonstrate that implementation support cannot safely be inferred
from implementation-specific code alone.

---

## 5. Trove classifiers

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

## Current limitation

The important question is not whether implementation identity can be written
down.

It can.

The question is whether the existing classifier vocabulary has sufficiently
precise semantics for machine-actionable compatibility decisions.

The current distinction under investigation is:

```text
classifier:

    descriptive classification

possible support metadata:

    normative release-level compatibility/support declaration
```

## Important adoption question

The ecosystem already has the ability to communicate implementation support,
but adoption appears uneven.

This creates an important challenge for a new field:

```text
Why would projects that do not reliably maintain
implementation classifiers maintain a new normative field?
```

A new mechanism would need a sufficiently valuable consumer benefit to justify
the additional maintenance burden.

## Evidence status

**Existing mechanism + adoption counter-evidence.**

Classifiers demonstrate that the concept is already expressible
descriptively.

They do not establish whether the ecosystem needs a second, normative
representation.

---

## 6. Sdist build avoidance

Packaging discussions have identified cases where an installer may select an
sdist and attempt a build that is likely to fail or is not intended for
ordinary installation.

This is broader than Python implementation compatibility.

The general problem is:

```text
source distribution
        ↓
unknown build characteristics
        ↓
expensive build attempt
        ↓
failure or unsuitable result
```

The research therefore considers whether pre-build knowledge could have
practical value.

## Relationship to implementation support

An implementation-specific build failure may arise from several different
causes:

```text
implementation bug
build dependency
host dependency
compiler/toolchain
ABI
private implementation usage
runtime support policy
```

These should not be collapsed into a single metadata concept.

In particular, PEP 725 is relevant to build and host requirements.

Therefore a build failure is not automatically evidence that a new
implementation-support field is necessary.

## Evidence status

**Relevant adjacent problem.**

It establishes the practical value of avoiding unnecessary source builds, but
does not establish that implementation support is the correct metadata layer.

---

## 7. Pure-Python detection discussion

A separate packaging discussion asked how tooling could determine whether an
sdist is pure Python without attempting a complete build.

This provides another example of a broader packaging problem:

```text
source distribution
        ↓
unknown build characteristics
        ↓
tool must perform work to discover them
```

## Why it matters to this research

This problem helps distinguish three separate concepts:

```text
implementation compatibility
        +
build characteristics
        +
artifact compatibility
```

They may interact, but they are not the same property.

For example:

```text
pure Python
```

does not necessarily mean:

```text
all Python implementations supported
```

Likewise:

```text
implementation-specific build
```

does not necessarily mean:

```text
implementation unsupported
```

## Research implication

A proposed implementation-support field should not become a general-purpose
replacement for build metadata.

Each piece of information should remain in the mechanism whose semantics match
the information being represented.

## Evidence status

**Adjacent evidence / boundary condition.**

---

## 8. Support inheritance and dependency graphs

Some implementation restrictions arise below the top-level package.

For example:

```text
package A
    |
    +--> dependency B
            |
            +--> CPython-specific component
```

It is unsafe to infer:

```text
B supports CPython only
        ↓
A supports CPython only
```

because A may:

* conditionally select B;
* provide an alternative dependency;
* provide a fallback implementation;
* use B only for an optional feature;
* or otherwise remain compatible with multiple implementations.

This is why packages such as Autobahn and wrapper-style projects are useful
control cases.

## Research implication

Implementation support is a property of the **release's effective behavior**,
not necessarily a property that can be derived transitively from one dependency.

This creates an important design requirement for any future metadata:

> A declaration must describe the support semantics intended for the release,
> rather than invite consumers to infer them transitively from dependency
> metadata.

## Evidence status

**Control / design constraint.**

---

# 9. Failed-build caching as an alternative

Another relevant observation from the packaging discussion is that tooling can
cache failed build results.

Conceptually:

```text
first attempt
    ↓
build fails
    ↓
cache failure
    ↓
avoid repeating identical work
```

This is relevant to the argument that metadata might be useful for avoiding
expensive failed source builds.

## Why caching is not equivalent to support metadata

A cached failure does not provide the same semantics as a producer declaration.

Caching:

```text
observed result in environment X
```

whereas support metadata would represent:

```text
producer declaration about release R
```

A cache also has limitations:

* the first attempt still occurs;
* the result may be environment-specific;
* a failure may be caused by a temporary issue;
* a later release may fix the problem;
* the result may not transfer safely between environments.

Therefore caching is a credible alternative for the **performance** aspect of
failed builds, but it does not by itself answer the semantic question of
whether a release supports an implementation.

## Evidence status

**Alternative / counterargument.**

This weakens the claim that a new field is automatically necessary merely
because source builds can fail.

---

# 10. Research interpretation

The current real-world evidence supports several observations:

* implementation-specific compatibility is real;
* implementation-specific build logic is real;
* some releases explicitly communicate implementation restrictions;
* generic wheel artifacts can coexist with narrower producer support statements;
* source distributions expose a release-level compatibility question that wheel
  tags do not always answer before a build;
* implementation information already exists descriptively through classifiers;
* packages can adapt to multiple implementations through fallbacks;
* implementation restrictions can arise from dependencies, ABI, build systems,
  private APIs, or alternative-interpreter limitations;
* failed-build caching provides an alternative for some performance-oriented
  cases.

The evidence does **not yet establish**:

* that implementation restrictions are common enough to justify new metadata;
* that all explicit CPython-only declarations represent technical
  incompatibility;
* that installers should reject unsupported implementations;
* that classifiers are insufficient for every consumer;
* that PEP 725 cannot address the relevant build cases;
* that a new Core Metadata field is preferable to index metadata, artifact
  metadata, or other mechanisms;
* that `Requires-Implementation` is the correct semantic model;
* or that `Supported-Implementation` is necessary.

---

# 11. Current evidence hierarchy

The cases in this document should be interpreted according to the following
rough hierarchy:

```text
Level 1 — normative specification
    ↓
Level 2 — published release metadata
    ↓
Level 3 — producer documentation
    ↓
Level 4 — source/runtime behavior
    ↓
Level 5 — community discussion
```

A producer statement is strong evidence that the producer intends a particular
support policy.

A runtime guard is strong evidence that a restriction is operationally
enforced.

Neither one alone proves that the restriction belongs in normative package
metadata.

The final standardization question requires evidence from all relevant layers.

---

# 12. Current conclusion

The real-world corpus establishes a meaningful phenomenon:

```text
release-level producer support
        can be narrower than
artifact-level compatibility metadata
```

The most interesting instances are releases where:

```text
producer:
    implementation-specific support boundary

artifact:
    implementation-generic wheel

sdist:
    available

Requires-Python:
    insufficient to express implementation identity
```

However, the research is not yet at the point where these observations should
be converted into a PEP recommendation.

The current working position is:

> There appears to be a genuine semantic distinction between artifact-level
> compatibility and producer-declared implementation support. Real releases
> demonstrate this distinction, but further evidence is required to determine
> whether it is sufficiently general, stable, and useful to justify a new
> normative metadata field.

The next step is therefore not to collect more examples indiscriminately.

It is to **adversarially validate the strongest examples**:

```text
real-world case
        ↓
root-cause classification
        ↓
existing mechanisms tested
        ↓
false-positive explanations eliminated
        ↓
actual consumer decision identified
        ↓
incremental benefit of new metadata measured
```

Only cases that survive this process should be promoted into
`evidence/residual-cases.md` as strong residual evidence.
