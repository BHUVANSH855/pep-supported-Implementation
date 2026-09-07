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

> **Where should that declaration live, what should its semantics be, and
> should installers use it during candidate selection?**

The repository therefore investigates the **problem first** and the field
design second.

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
    ← wheel compatibility tags

ENVIRONMENT
    "Which implementation / ABI / platform does this environment provide?"
    ← sys.implementation / PEP 508 / ABI metadata

BUILD REQUIREMENTS
    "What environment is required to build this source?"
    ← PEP 517 / PEP 725 and related mechanisms
```

These are related but **not equivalent statements**.

The strongest recurring empirical pattern found so far is:

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

normative release-level implementation constraint:
    absent
```

This is a genuine packaging phenomenon.

It is **not yet proof that a new Core Metadata field is necessary**.

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

> "Python packaging has no implementation metadata."

It clearly does.

The narrower question is whether there is a missing **semantic layer for
producer-declared release support**.

---

# Four concepts that must not be conflated

A major result of the investigation is that the following concepts are
different:

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

What the maintainer is willing to support for a release.

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

These should not be collapsed into one metadata mechanism.

---

# Why the sdist case matters

Wheel compatibility is already machine-actionable.

A wheel such as:

```text
foo-1.0-cp313-cp313-linux_x86_64.whl
```

contains implementation information in its compatibility tags.

An implementation that cannot use that wheel can reject it before installation.

An implementation-generic wheel such as:

```text
foo-1.0-py3-none-any.whl
```

does not make the same implementation-specific claim.

The difficult case is an sdist:

```text
foo-1.0.tar.gz
```

When an installer has no compatible wheel, it may consider the source
distribution and build a wheel locally.

The research question is therefore:

> **Can an installer know from standardized release metadata that the source
> distribution is not supported by the current Python implementation before
> starting the build?**

At present, there is no dedicated release-level implementation constraint
equivalent to the implementation component of wheel tags.

That is the clearest unresolved gap identified by this research.

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

Therefore PEP 508 is important infrastructure for this research, but it is
not currently equivalent to a release-support declaration.

---

# What wheel tags do and do not solve

Wheel tags are the established artifact-level compatibility mechanism.

For example:

```text
cp313-cp313-linux_x86_64
```

can communicate that a particular wheel is built for CPython.

This is excellent for **built artifact selection**.

However, a release can simultaneously contain:

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
producer support policy
```

The research is therefore not proposing to replace wheel tags.

---

# Strong empirical cases

The current corpus contains several useful cases.

These are evidence, not proof that a new standard is necessary.

---

## RestrictedPython 8.5

PyPI publishes:

```text
restrictedpython-8.5.tar.gz
restrictedpython-8.5-py3-none-any.whl
```

and the release metadata includes:

```text
Programming Language :: Python :: Implementation :: CPython
```

The project explicitly states that RestrictedPython supports CPython and does
not support PyPy or other Python implementations.

This is an especially useful case because:

```text
wheel:
    py3-none-any

producer support:
    CPython only
```

The case demonstrates a real difference between artifact tagging and producer
support.

### Important limitation

This example does **not** prove that a new Core Metadata field is the correct
solution.

It could instead indicate that the published wheel is too broadly tagged.

The research must therefore treat it as:

```text
evidence of a support/artifact mismatch
```

rather than:

```text
proof that Requires-Implementation is required
```

---

## HAX 0.3.0

HAX publishes:

```text
hax-0.3.0.tar.gz
hax-0.3.0-py3-none-any.whl
```

and explicitly describes support in CPython terms.

Its source contains an implementation check equivalent to:

```python
if implementation.name != "cpython":
    raise RuntimeError("HAX only supports CPython!")
```

This is particularly valuable because the restriction is not merely a
classifier or documentation statement.

There is:

```text
documentation
+
published artifact
+
source-level enforcement
```

The case therefore demonstrates a genuine implementation-specific semantic
restriction that is not represented by `Requires-Python`.

---

## Likepy 0.3.0

PyPI publishes:

```text
likepy-0.3.0.tar.gz
likepy-0.3.0-py3-none-any.whl
```

with a CPython implementation classifier and a description stating that only
CPython is supported.

This independently reproduces the same general pattern:

```text
implementation-specific producer support
+
implementation-generic Python wheel
+
sdist
```

It is useful because it is independent of the RestrictedPython example.

---

## simple-ctx-log 0.0.3

This 2026 release publishes:

```text
simple_ctx_log-0.0.3.tar.gz
simple_ctx_log-0.0.3-py3-none-any.whl
```

and documents use of:

```text
sys._getframe
```

as a CPython-only feature.

This is useful as a recent independent example of:

```text
pure Python
+
implementation-specific semantics
+
py3-none-any
+
sdist
```

The repository should remain conservative here: published documentation is
strong evidence, but this case should not be described as source-enforced
unless independent source evidence confirms that.

---

## TribeCore 4.7.3

This 2026 release explicitly states:

```text
CPython only.
PyPy and other Python implementations are not supported.
```

It publishes an sdist and wheels with Python tags of the form:

```text
py3-none-<platform>
```

The project explains that the generic Python tag allows the same wheel to work
across Python 3.x versions for the target platform.

This is valuable because the phenomenon is not limited to:

```text
py3-none-any
```

It can also occur with platform-specific wheels whose **Python implementation
tag remains generic**.

### Caveat

TribeCore includes bundled/native components, so its build and artifact layers
need separate analysis before treating it as a clean pure-Python residual case.

---

# Cases that must NOT be overinterpreted

Some projects are valuable precisely because they show where inference fails.

## psutil

Implementation-specific build logic does not automatically imply
implementation-specific support.

A project can have CPython-specific paths while supporting multiple Python
implementations.

Therefore:

```text
implementation-specific source code
```

does not imply:

```text
implementation unsupported
```

---

## Autobahn

A project can support multiple Python implementations while some optional or
native dependencies have narrower implementation support.

Therefore:

```text
CPython-only dependency
```

does not imply:

```text
CPython-only release
```

Implementation support cannot safely be inferred transitively.

---

## Guppy3

Guppy3 is an excellent implementation/ABI case.

Its CPython-specific wheels already encode substantial artifact-level
compatibility information.

Therefore it is especially useful for studying:

```text
implementation identity
+
ABI
+
Python version
+
wheel compatibility
+
source builds
```

but it is a weaker example for demonstrating a residual **release-level**
metadata gap.

---

# The strongest counterargument

The strongest counterargument is not:

> "implementation compatibility doesn't exist."

It clearly does.

The strongest counterargument is:

> **The existing ecosystem may already contain enough information for the
> practical cases, and the remaining cases may not justify a new normative
> metadata field.**

Possible existing solutions include:

* correcting incorrectly tagged wheels;
* improving classifiers;
* improving installer diagnostics;
* using source-build policy;
* extending PEP 725;
* improving index metadata;
* richer wheel variants;
* doing nothing until a larger corpus demonstrates demand.

The research must test these alternatives fairly.

---

# PEP 725 is a first-class alternative

PEP 725 must not be treated as a footnote.

It addresses external dependencies and distinguishes build, host, and runtime
requirements.

That makes it directly relevant to the question:

> Is the real problem actually a build-environment requirement?

However, the following statements are not necessarily equivalent:

```text
The package must be built under CPython.
```

and:

```text
The resulting release only supports CPython.
```

For example, a project might:

```text
build under CPython
        ↓
produce py3-none-any
        ↓
run on PyPy
```

Therefore:

```text
build implementation
```

must not automatically become:

```text
runtime implementation
```

The research must determine whether PEP 725 can represent the actual residual
cases without conflating these layers.

---

# The central design question

If a new field is eventually justified, its semantics need to be chosen
carefully.

The original proposal was:

```text
Requires-Implementation: cpython
```

But this terminology may be misleading.

It sounds like:

```text
technical requirement
```

rather than:

```text
producer support declaration
```

A potentially more accurate concept is:

```text
Supported-Implementation: cpython
```

or an equivalent release-support mechanism.

This is not a recommendation yet.

It is a hypothesis that needs ecosystem validation.

---

# Why positive support may be preferable

Compare:

```text
Requires-Implementation: cpython
```

with:

```text
Supported-Implementation: cpython
```

The first naturally suggests:

```text
anything else is impossible
```

The second can mean:

```text
this is an implementation the producer explicitly supports
```

That distinction matters because alternative implementations can gain support
over time.

A package that says:

```text
CPython only
```

today might support PyPy next year.

A positive support declaration can therefore be less semantically dangerous
than a permanent negative compatibility claim.

However, this immediately raises another question:

> What does omission mean?

The repository does not currently assume that omission means unsupported.

For backwards compatibility, the safest interpretation may be:

```text
field absent
    =
no normative support declaration
```

rather than:

```text
field absent
    =
all other implementations unsupported
```

---

# Support must not be inferred

The research deliberately rejects several tempting inference rules.

## No PyPy wheel

Does not prove:

```text
PyPy unsupported
```

A project may support PyPy from source without publishing a dedicated wheel.

## CPython classifier

Does not prove:

```text
all other implementations unsupported
```

A classifier is not necessarily an exhaustive compatibility set.

## `sys.implementation` usage

Does not prove:

```text
CPython only
```

The code may have a fallback.

## Native extension

Does not automatically justify new metadata.

Wheel tags may already solve the relevant artifact-selection problem.

## CPython-only dependency

Does not automatically make the parent package CPython-only.

The parent may provide an alternative implementation path.

---

# Candidate solutions

The research currently compares these possibilities.

## 1. Keep classifiers descriptive

```text
Programming Language :: Python :: Implementation :: CPython
```

Pros:

* no new metadata;
* existing vocabulary;
* already widely understood.

Cons:

* no normative installer behavior;
* absence is ambiguous;
* not necessarily exhaustive.

---

## 2. Give classifiers normative semantics

Pros:

* no new field.

Cons:

* changes the meaning of existing metadata;
* old projects may suddenly become incompatible;
* classifier absence was not designed as a hard compatibility assertion.

This has significant backwards-compatibility risk.

---

## 3. `Requires-Implementation`

```text
Requires-Implementation: cpython
```

Pros:

* familiar requirement-style model;
* easy to explain;
* potentially useful for candidate selection.

Cons:

* conflates support with technical requirement;
* can become stale;
* unclear interaction with build requirements;
* needs precise semantics for implementation forks and variants.

---

## 4. `Supported-Implementation`

```text
Supported-Implementation: cpython
Supported-Implementation: pypy
```

Pros:

* describes producer support;
* supports multiple implementations;
* avoids saying unlisted implementations are impossible.

Cons:

* still requires normative semantics;
* omission must be defined;
* incorrect declarations could cause false negatives;
* "support" itself needs a precise definition.

This is currently the **most semantically attractive hypothesis**, not an
established recommendation.

---

## 5. Wheel tags

Keep using wheel tags for artifact compatibility.

This is unquestionably valuable and should not be replaced.

---

## 6. PEP 825 / wheel variants

Potentially useful for richer artifact compatibility.

Not obviously a replacement for a release-level support declaration.

---

## 7. PEP 508 markers

Excellent for conditional dependencies.

Not currently a mechanism for saying:

```text
reject this distribution itself
```

---

## 8. PEP 780

Useful for ABI feature compatibility.

It should remain separate from implementation support.

---

## 9. PEP 725

Important for build/host/external dependency semantics.

Requires deeper investigation before deciding whether it can subsume the
residual problem.

---

## 10. Source-build policy

A mechanism such as:

```text
do not automatically build this sdist
```

could solve some operational failures.

But it answers:

```text
should the installer build this source?
```

rather than:

```text
which implementations does this release support?
```

These are related but different questions.

---

## 11. No new standard

This remains a completely valid outcome.

If existing mechanisms plus tooling improvements solve the meaningful cases,
standardization should stop.

---

# Current evidence matrix

| Mechanism                  |         Release support |       Sdist |  Resolver use | Existing adoption | Main problem                  |
| -------------------------- | ----------------------: | ----------: | ------------: | ----------------: | ----------------------------- |
| Trove classifiers          |      Yes, descriptively |         Yes |           Low |              High | Not normative                 |
| Reinterpreted classifiers  |             Potentially |         Yes |          High |              High | Backwards compatibility       |
| `Requires-Implementation`  |                     Yes |         Yes |          High |              None | Requirement/support ambiguity |
| `Supported-Implementation` |                     Yes |         Yes |          High |              None | Definition of support         |
| Wheel tags                 |           Artifact only |          No |          High |              High | Not release-level             |
| PEP 825                    |        Artifact/variant |     Limited |          High |          Emerging | Not support policy            |
| PEP 508                    |            Dependencies |     Partial | High for deps |              High | Not self-compatibility        |
| PEP 780                    |                     ABI |     Partial |  High for ABI |          Emerging | Different dimension           |
| PEP 725                    | Build/host/runtime deps | Potentially |      Emerging |          Emerging | Not currently release support |
| Source-build policy        |             Operational |         Yes |   Potentially |               Low | Does not describe support     |
| No new standard            |         Existing layers |     Partial |       Partial |           Highest | Residual cases remain         |

This matrix is a research instrument, not a final recommendation.

---

# Evidence standard

The repository distinguishes three things:

### Direct evidence

Examples:

* normative packaging specifications;
* published PyPI metadata;
* published filenames;
* project documentation;
* source-level compatibility checks;
* actual standards discussions.

### Interpretation

A reasoned mapping from those facts to the research question.

### Hypothesis

A proposed mechanism that still needs validation.

For example:

```text
No PyPy wheel
```

must not become:

```text
PyPy unsupported
```

without additional evidence.

Likewise:

```text
sys.implementation.name is inspected
```

must not automatically become:

```text
package is CPython-only
```

---

# What would falsify the proposal?

The research should conclude against a new field if evidence shows that:

1. existing classifiers can safely acquire the necessary semantics;
2. wheel/index metadata already provides the required release-level information;
3. PEP 725 can represent the meaningful residual cases without semantic
   conflation;
4. a source-build policy solves the actual user-facing problem;
5. the remaining cases are rare or operationally insignificant;
6. consumers do not need the information before building/installing;
7. the field cannot define "support" precisely enough for automated use;
8. the maintenance cost exceeds the resolver/tooling benefit.

A negative conclusion is a successful research result.

---

# What would strengthen the proposal?

The proposal becomes substantially stronger if we can demonstrate all of the
following:

```text
1. A concrete class of releases is genuinely implementation-restricted.

2. The restriction is producer-declared or otherwise directly evidenced.

3. The release has an sdist.

4. Existing Requires-Python cannot express it.

5. PEP 508 markers cannot express the package's own restriction.

6. Wheel tags cannot help because no compatible wheel exists yet.

7. The installer therefore has to consider/build the sdist.

8. The installer could have rejected the candidate earlier if a normative
   release-level declaration existed.

9. PEP 725 and other existing proposals cannot express the same semantic fact
   cleanly.

10. Real users/tools would benefit from acting on that information.
```

That is the evidence threshold the research should aim for.

---

# Next research phase

The repository should now move from example collection to **adversarial
validation**.

## Phase 1 — Freeze the corpus

For every candidate release record:

```text
project
version
release date
Requires-Python
classifiers
sdist filename
wheel filenames
implementation statement
source/runtime evidence
dependency markers
ABI evidence
```

---

## Phase 2 — Separate the compatibility layers

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

Do not use one as a proxy for another.

---

## Phase 3 — Test actual candidate selection

The key experiment is:

```text
current implementation
        ↓
pip / resolver
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

> **At what point could the installer have known that the release was not
> supported?**

---

## Phase 4 — Test PEP 725 explicitly

For every strong residual case ask:

```text
Can PEP 725 express the actual requirement?

If yes:
    Does it describe build compatibility only?

If no:
    What exact semantic information is missing?
```

Do not argue against PEP 725 abstractly.

Demonstrate the mismatch with concrete cases.

---

## Phase 5 — Identify consumers

A metadata field has value only if somebody can act on it.

Potential consumers include:

* pip;
* uv;
* Poetry;
* package indexes;
* dependency resolvers;
* build frontends;
* vulnerability/scanning tools;
* environment managers;
* IDEs and package browsers.

The research should establish which of these actually need the information.

---

# Current conclusion

The research has reached a useful intermediate conclusion:

> **Python packaging already has strong artifact-level implementation
> compatibility through wheel tags and environment-level implementation
> information through `sys.implementation` and PEP 508. The unresolved area
> is producer-declared release-level implementation support, particularly when
> an installer is considering an sdist before a compatible wheel exists.**

That is a much narrower and more defensible statement than:

> "Python needs `Requires-Implementation`."

The repository should therefore remain **pre-PEP and design-neutral** until the
remaining empirical questions are answered.

The next objective is not to prove a field is necessary.

It is to determine whether a **real residual problem remains after accounting
for wheel tags, classifiers, PEP 508, PEP 725, ABI metadata, wheel variants,
and source-build policy**.

If that residual problem is real, common, and actionable, then a new standard
may be justified.

If not, the correct result is to document why existing mechanisms are enough.

---

## Primary resources

* Python Packaging User Guide specifications
* Core Metadata specification
* Source Distribution specification
* Wheel/platform compatibility specifications
* PEP 421 — `sys.implementation`
* PEP 425 — Compatibility Tags for Built Distributions
* PEP 508 — Dependency Specification
* PEP 621 — Project Metadata
* PEP 625 — Source Distribution File Name
* PEP 643 — Metadata 2.2
* PEP 658 / PEP 714 — Core Metadata through the Simple API
* PEP 725 — External Dependencies
* PEP 780 — ABI Feature Environment Markers
* PEP 825 — Wheel Variants
* Trove classifiers
* Python.org Packaging discussions
* PyPI release metadata and artifacts
* PyPy compatibility documentation

---

## Repository discipline

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

The goal is a standards-quality empirical investigation that remains useful
even if the eventual answer is:

> **No new metadata field is needed.**
