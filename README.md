# Research: Python Implementation Support Metadata

Research, prior art, and empirical evidence for a possible Python packaging
standard for declaring **release-level Python implementation support**.

> **Current status (2026-09-07): research / pre-PEP.**
>
> This repository does **not** claim that a new Core Metadata field is
> necessary. It investigates whether Python packaging has a meaningful
> release-level implementation-support gap that existing metadata and artifact
> mechanisms cannot represent safely.

---

## Research question

The original proposal was a Core Metadata field named:

```text
Requires-Implementation
```

The research has deliberately moved to a broader and more neutral question:

> **Does Python packaging need a normative release-level declaration of
> supported Python implementations, distinct from descriptive implementation
> classifiers and artifact compatibility metadata?**

If the answer is yes, a second question follows:

> **Where should that declaration live, what should its semantics be, and should
> installers or other consumers use it during candidate selection?**

The repository therefore investigates the **problem first** and the field
design second.

The project is explicitly willing to conclude:

> **No new metadata field is needed.**

---

# Executive finding

The research currently establishes a real distinction between several layers of
Python compatibility information.

```text
PROJECT / RELEASE SUPPORT

    "Which Python implementations does the producer support?"

    ← research question


ARTIFACT COMPATIBILITY

    "Can this particular built artifact run here?"

    ← wheel compatibility tags / variants


ENVIRONMENT

    "Which implementation / ABI / platform does this environment provide?"

    ← sys.implementation / environment markers / ABI information


BUILD REQUIREMENTS

    "What environment is required to build this source?"

    ← PEP 517 / PEP 725 and related mechanisms
```

These are related but **not equivalent statements**.

A major result of the investigation is that the following concepts must remain
separate:

```text
requirement
artifact compatibility
producer support
observed compatibility
support policy
evidence
```

The distinction is important because:

```text
observed restriction
        ≠
root cause
        ≠
metadata gap
        ≠
need for a new standard
```

---

# The strongest empirical pattern

The current corpus contains releases with patterns such as:

```text
producer:

    CPython only

release:

    sdist available

wheel:

    py3-... implementation-generic Python tag

Requires-Python:

    version constraint only

classifier:

    CPython implementation classifier

normative release-level implementation support:

    absent
```

This is a genuine packaging phenomenon.

However:

> **It is not yet proof that a new Core Metadata field is necessary.**

The research now requires each candidate to survive root-cause and
consumer-benefit analysis before being promoted to strong residual evidence.

---

# What existing packaging already solves

Python packaging already has several implementation and compatibility
mechanisms.

| Question                                                  | Existing mechanism           | Current role                           |
| --------------------------------------------------------- | ---------------------------- | -------------------------------------- |
| Which Python versions are required?                       | `Requires-Python`            | Normative / installer-relevant         |
| Which implementation is running?                          | `sys.implementation`         | Runtime environment                    |
| Which implementation is present in a marker?              | `implementation_name`        | PEP 508 dependency conditions          |
| Which implementation does a wheel target?                 | Wheel Python tag             | Artifact compatibility                 |
| Which ABI does a wheel target?                            | Wheel ABI tag / ABI metadata | Artifact compatibility                 |
| Which platform does a wheel target?                       | Wheel platform tag           | Artifact compatibility                 |
| Which ABI features exist?                                 | PEP 780                      | Environment/dependency compatibility   |
| Which wheel variants are available?                       | PEP 825 work                 | Artifact/index compatibility           |
| Which external build/host dependencies exist?             | PEP 725                      | Build/host/runtime dependency metadata |
| Which implementations does a project mention?             | Trove classifiers            | Descriptive                            |
| Which implementations does a release normatively support? | —                            | **Open research question**             |

Therefore the proposal should **never** be described as:

> “Python packaging has no implementation metadata.”

It clearly does.

The narrower question is whether there is a missing **semantic layer for
producer-declared release support**.

---

# Four compatibility concepts

A major result of the investigation is that these concepts should not be
conflated.

## Requirement

What the software technically needs.

Example:

```text
requires CPython internals
```

## Artifact compatibility

What a particular built artifact can run on.

Example:

```text
cp313-cp313-manylinux_2_28_x86_64
```

## Producer support

What the maintainer explicitly supports for a release.

Example:

```text
CPython only
```

## Evidence

Why we believe the support claim.

Examples:

```text
CI matrix
runtime guard
source inspection
published wheels
documentation
actual tests
```

A support declaration is therefore not automatically a technical requirement,
and a technical restriction is not automatically a reason for a new metadata
field.

---

# Root-cause analysis

A statement such as:

```text
CPython only
```

does not by itself explain why.

The repository classifies implementation restrictions using the following
taxonomy:

| Class | Root cause                                  |
| ----- | ------------------------------------------- |
| A     | Runtime semantic restriction                |
| B     | Build/toolchain restriction                 |
| C     | Alternative-implementation bug/workaround   |
| D     | ABI/configuration restriction               |
| E     | Private implementation usage                |
| F     | Explicit support-policy declaration         |
| G     | Conditional/fallback implementation support |
| H     | Dependency/component restriction            |

This distinction is essential.

For example:

```text
package fails to build on PyPy
```

could result from:

```text
alternative interpreter bug
```

rather than:

```text
package intentionally supports CPython only
```

Similarly:

```text
package uses sys.implementation
```

does not automatically mean:

```text
package is CPython-only
```

The full methodology is documented in:

`evidence/root-cause-taxonomy.md`

---

# Why the sdist case matters

Wheel compatibility is already machine-actionable.

A wheel such as:

```text
foo-1.0-cp313-cp313-linux_x86_64.whl
```

contains implementation information in its compatibility tags.

An implementation that cannot use that wheel can reject it before installation.

The difficult case is:

```text
foo-1.0.tar.gz
```

When no compatible wheel exists, an installer may consider the source
distribution and build a wheel locally.

The central research question is therefore:

> **Can a consumer know from standardized release metadata that an sdist is not
> supported by the current Python implementation before starting the build?**

At present, there is no dedicated release-level implementation constraint
directly analogous to the implementation component of wheel compatibility
tags.

That is the clearest unresolved gap identified so far.

But the existence of this gap does not establish that a new field is the
correct solution.

---

# What `Requires-Python` does not solve

`Requires-Python` describes Python **version compatibility**.

For example:

```text
Requires-Python: >=3.10,<3.16
```

does not distinguish:

```text
CPython 3.13
PyPy 3.13
GraalPy 3.13
```

Therefore:

```text
Python version compatibility
```

and:

```text
Python implementation compatibility
```

are separate dimensions.

This does not mean `Requires-Python` should be extended to encode arbitrary
implementation logic.

---

# What PEP 508 markers do not solve

PEP 508 provides environment markers such as:

```text
implementation_name == "cpython"
```

These are useful for conditional dependencies:

```text
Requires-Dist: package-x; implementation_name == "cpython"
```

But this answers:

> When should dependency X be installed?

It does not directly answer:

> Is this distribution itself a valid candidate for the current
> implementation?

Therefore PEP 508 is important infrastructure for this research, but is not
currently equivalent to a release-support declaration.

---

# What wheel tags do and do not solve

Wheel tags are the established artifact-level compatibility mechanism.

For example:

```text
cp313-cp313-linux_x86_64
```

can communicate that a particular wheel is built for CPython.

This is excellent for **built artifact selection**.

However, a release can contain:

```text
foo-1.0-py3-none-any.whl
foo-1.0.tar.gz
```

while its producer documentation says:

```text
CPython only
```

This creates the central distinction:

```text
artifact compatibility
        ≠
producer support
```

The research is therefore not proposing to replace wheel tags.

---

# Real-world evidence

The repository deliberately separates broad real-world observations from
surviving residual cases.

## Broad evidence

`evidence/real-world-cases.md` records:

* implementation-specific packages;
* adaptive/fallback packages;
* classifiers;
* build-avoidance discussions;
* tooling use cases;
* dependency/component restrictions;
* alternative mechanisms;
* counterexamples.

These establish that implementation compatibility is a real ecosystem concern.

## Residual evidence

`evidence/residual-cases.md` applies stricter filtering.

A case should become a strong residual case only if the research can establish:

```text
explicit producer support boundary
        ↓
concrete release
        ↓
release-level candidate
        ↓
existing artifact metadata insufficient
        ↓
Requires-Python insufficient
        ↓
PEP 508 insufficient for self-support
        ↓
ABI mechanisms insufficient
        ↓
root cause understood
        ↓
alternative explanations eliminated
        ↓
meaningful consumer decision exists
        ↓
new metadata provides incremental value
```

This prevents the repository from counting every “CPython only” statement as
evidence for a new standard.

---

# Current candidate cases

The current corpus includes releases such as:

* RestrictedPython 8.5;
* HAX 0.3.0;
* Likepy 0.3.0;
* simple-ctx-log 0.0.3;
* TribeCore 4.7.3.

These cases demonstrate the general phenomenon but are not all equally strong.

The current strongest candidate is **HAX 0.3.0**, because its CPython
restriction is explicitly enforced at runtime while the published wheel is
implementation-generic.

Other cases require additional root-cause and consumer-benefit validation.

Control cases include projects such as:

* psutil;
* Autobahn;
* Guppy3;
* bocpy.

These are important because they demonstrate that:

```text
implementation-specific code
        ≠
implementation unsupported
```

and:

```text
CPython-specific dependency
        ≠
CPython-only top-level release
```

---

# Requirement vs support

The original proposal was:

```text
Requires-Implementation: cpython
```

This sounds like a hard technical requirement.

A potentially different concept is:

```text
Supported-Implementation: cpython
```

which sounds like a producer support declaration.

The distinction is:

```text
Requires-Implementation
    ↓
environment must satisfy this condition

Supported-Implementation
    ↓
producer explicitly supports this implementation
```

The current research considers positive support semantics more promising than
turning implementation identity directly into a hard requirement.

However:

> **`Supported-Implementation` is a hypothesis, not a recommendation.**

The repository will not freeze the field name until the underlying problem and
consumer semantics are established.

---

# The central semantic problem

Suppose a release declares:

```text
Supported-Implementation: cpython
```

What does that mean for PyPy?

There are several possibilities.

### Exhaustive semantics

Only listed implementations are supported.

```text
CPython → supported
PyPy    → unsupported
```

This provides strong candidate filtering but turns omission into a negative
claim.

### Positive semantics

Listed implementations are explicitly supported.

```text
CPython → explicitly supported
PyPy    → unknown
```

This is safer but provides weaker candidate filtering.

The repository currently considers the positive interpretation the safer
hypothesis.

This remains unresolved.

---

# What absence should mean

The research currently prefers:

```text
field absent
    =
no normative implementation-support declaration
```

rather than:

```text
field absent
    =
all implementations supported
```

or:

```text
field absent
    =
all unlisted implementations unsupported
```

This is important for backwards compatibility.

Existing distributions should not acquire a new compatibility meaning merely
because they predate a future metadata field.

---

# Important non-inferences

The research explicitly rejects several tempting inference rules.

## No PyPy wheel

```text
No PyPy wheel
    ≠
PyPy unsupported
```

A project may support PyPy from source without publishing a dedicated wheel.

## CPython classifier

```text
CPython classifier
    ≠
all other implementations unsupported
```

A classifier is descriptive unless a future specification gives it stronger
semantics.

## `sys.implementation` usage

```text
sys.implementation usage
    ≠
CPython-only package
```

The code may have fallbacks or conditional behavior.

## Native extension

```text
native extension
    ≠
new implementation metadata required
```

Wheel tags or ABI metadata may already represent the relevant compatibility.

## CPython-only dependency

```text
CPython-only dependency
    ≠
CPython-only release
```

The parent project may provide a fallback or conditional dependency path.

---

# Alternatives under investigation

A new Core Metadata field is only one candidate.

The repository currently compares:

1. existing descriptive classifiers;
2. normative classifier semantics;
3. `Requires-Implementation`;
4. `Supported-Implementation`;
5. wheel tags;
6. PEP 825 wheel variants;
7. PEP 508 markers;
8. PEP 780 ABI features;
9. PEP 725 external/build/host dependencies;
10. source-build policy;
11. failed-build caching;
12. no new standard.

Each mechanism is evaluated against the actual residual problem rather than
against an abstract proposal.

---

# PEP 725 is a first-class alternative

PEP 725 is directly relevant whenever an apparent implementation restriction
is actually a build or host requirement.

These statements must not be conflated:

```text
The package must be built under CPython.
```

and:

```text
The resulting release only supports CPython.
```

For example:

```text
build under CPython
        ↓
produce py3-none-any
        ↓
run on PyPy
```

is possible in principle.

Therefore:

```text
build implementation
        ≠
runtime implementation
```

The research must test PEP 725 against concrete residual cases rather than
arguing against it abstractly.

---

# Failed-build caching is also relevant

Tooling can cache failed build results:

```text
first attempt
    ↓
build fails
    ↓
cache result
    ↓
avoid repeating equivalent work
```

This can address some performance-oriented source-build problems.

However:

```text
cached failure
    ≠
producer support declaration
```

A cache records an observed result in an environment.

Support metadata would represent a producer declaration about a release.

The research therefore treats caching as a legitimate alternative for some
operational problems, but not as a semantic replacement for support metadata.

---

# Core Metadata vs other layers

If the underlying problem survives existing mechanisms, the next question is
where the information belongs.

Possible locations include:

```text
Core Metadata
index/repository metadata
wheel metadata/tags
wheel variants
build metadata
project documentation
tool-specific configuration
```

Core Metadata is attractive because a release-level support declaration could
travel with the distribution.

But adding Core Metadata has costs:

* specification complexity;
* build-backend support;
* metadata validation;
* installer behavior;
* repository behavior;
* documentation;
* maintenance;
* stale declarations;
* ecosystem adoption.

The repository therefore treats Core Metadata as a **candidate**, not a
premise.

---

# What would falsify the proposal?

The research should conclude against a new field if evidence shows that:

1. existing classifiers provide sufficient information for meaningful consumers;
2. wheel/index metadata already provides the required information;
3. PEP 725 adequately represents the meaningful build cases;
4. source-build policy solves the actual operational problem;
5. failed-build caching provides most of the claimed practical benefit;
6. the remaining cases are rare or operationally insignificant;
7. consumers do not need the information before building/installing;
8. producer declarations cannot be defined precisely enough for automation;
9. stale declarations create unacceptable false negatives;
10. ecosystem maintenance costs exceed the consumer benefit.

A negative conclusion is a successful research result.

---

# What would strengthen the proposal?

The proposal becomes substantially stronger if investigation demonstrates:

```text
1. A concrete class of releases is genuinely implementation-restricted.

2. The restriction is producer-declared or directly evidenced.

3. The release has an sdist or equivalent release-level candidate.

4. Existing Requires-Python cannot express the restriction.

5. PEP 508 cannot express the distribution's own restriction.

6. Wheel tags cannot solve the problem because the relevant artifact does not
   yet exist.

7. The root cause is not merely ABI, build tooling, dependency structure,
   private implementation usage, or an alternative-interpreter bug.

8. The consumer therefore faces a meaningful pre-install/build decision.

9. Existing mechanisms cannot adequately express the required information.

10. A machine-readable release-level declaration materially improves that
    decision.

11. The benefit is large enough to justify specification and ecosystem costs.
```

That is the evidence threshold the research should aim for.

---

# Research methodology

The repository follows:

```text
evidence
    ↓
root-cause classification
    ↓
existing-mechanism analysis
    ↓
residual-case validation
    ↓
consumer decision
    ↓
design hypothesis
    ↓
standardization decision
```

It deliberately avoids:

```text
hypothesis
    ↓
selective examples
    ↓
conclusion
```

The methodology is documented in:

* `evidence/methodology.md`
* `evidence/root-cause-taxonomy.md`
* `evidence/real-world-cases.md`
* `evidence/residual-cases.md`

---

# Research architecture

The repository separates the investigation into layers.

```text
prior-art/
    ↓
What existing specifications already provide

evidence/real-world-cases.md
    ↓
What exists in the ecosystem

evidence/root-cause-taxonomy.md
    ↓
Why those restrictions exist

evidence/residual-cases.md
    ↓
Which cases survive adversarial filtering

evidence/methodology.md
    ↓
How evidence is evaluated

design/requirements-vs-supported.md
    ↓
What the possible semantics actually mean

design/decision-matrix.md
    ↓
Which mechanisms can solve the surviving problem

design/open-questions.md
    ↓
What remains unresolved

research-status.md
    ↓
Current overall conclusion
```

This separation is intentional.

---

# Current research status

The current position is:

```text
Problem exists?
    → likely yes

Semantic mismatch demonstrated?
    → yes

Strong residual cases?
    → at least one promising case; broader validation ongoing

Root causes fully classified?
    → no

Existing mechanisms fully eliminated?
    → no

Consumer demand demonstrated?
    → not yet sufficiently

New Core Metadata field justified?
    → not established

Requires-Implementation preferred?
    → no

Supported-Implementation preferred?
    → promising hypothesis, not established

No-new-standard outcome?
    → fully viable
```

The research confidence is therefore deliberately asymmetric:

> There is substantial evidence for a **real semantic distinction**, but much
> less evidence that the distinction requires a **new normative Core Metadata
> field**.

---

# Immediate next research phase

The next phase is **adversarial validation**, not proposal drafting.

## 1. Freeze the strongest cases

For each candidate release record:

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
root cause
```

## 2. Separate compatibility layers

For every case determine independently:

```text
Python version
implementation identity
ABI
platform
build environment
runtime environment
artifact compatibility
producer support policy
```

Do not use one dimension as a proxy for another.

## 3. Test actual candidate selection

The key experiment is:

```text
current implementation
        ↓
resolver
        ↓
candidate selection
        ↓
sdist
        ↓
build
        ↓
failure or success
```

The important question is:

> **At what point could the consumer have known that the release was not
> supported?**

## 4. Test PEP 725 concretely

For every strong candidate ask:

```text
Can PEP 725 express the actual requirement?

If yes:
    Is it only a build/host requirement?

If no:
    What exact semantic information is missing?
```

## 5. Identify actual consumers

Potential consumers include:

* pip;
* uv;
* Poetry;
* package indexes;
* dependency resolvers;
* build frontends;
* compatibility test systems;
* fuzzing infrastructure;
* environment managers;
* IDEs and package browsers.

The research should establish which consumers actually need the information.

---

# Primary resources

The repository tracks and analyzes:

* Python Packaging specifications;
* Core Metadata;
* Source Distribution specifications;
* wheel compatibility specifications;
* PEP 421 — `sys.implementation`;
* PEP 425 — Compatibility Tags for Built Distributions;
* PEP 508 — Dependency Specification;
* PEP 621 — Project Metadata;
* PEP 625 — Source Distribution File Name;
* PEP 643 — Metadata 2.2;
* PEP 658 / PEP 714 — Core Metadata through the Simple API;
* PEP 725 — External Dependencies;
* PEP 780 — ABI Feature Environment Markers;
* PEP 825 — Wheel Variants;
* Trove classifiers;
* Python.org Packaging discussions;
* PyPI release metadata and artifacts;
* Python implementation compatibility documentation.

See `SOURCES.md` for the maintained source inventory.

---

# Repository discipline

This repository should continue to prefer:

```text
evidence
    ↓
interpretation
    ↓
hypothesis
```

rather than:

```text
hypothesis
    ↓
selective examples
    ↓
conclusion
```

In particular:

```text
No PyPy wheel
    ≠
PyPy unsupported
```

```text
CPython classifier
    ≠
formal incompatibility declaration
```

```text
sys.implementation check
    ≠
proof that a new metadata field is required
```

```text
build failure
    ≠
release-level implementation requirement
```

The goal is a standards-quality empirical investigation that remains useful
even if the eventual answer is:

> **No new metadata field is needed.**
