# Open Questions

This document deliberately records questions that the research has not
resolved.

The questions are ordered from the underlying problem to the possible metadata
design. This is intentional: the research should establish that a
standardization problem survives existing mechanisms **before** optimizing the
syntax or semantics of a proposed field.

The central distinction throughout this document is:

```text id="k0f1da"
observed implementation restriction
        ≠
root cause of restriction
        ≠
metadata representation problem
        ≠
need for a new standard
```

---

## 1. Does a genuine release-level support problem remain?

The research has identified releases where:

```text id="bq1j9x"
producer:

    CPython only

artifact:

    py3-...

sdist:

    present

Requires-Python:

    version restriction only

classifier:

    descriptive implementation information
```

This establishes a semantic mismatch worth investigating.

The unresolved question is whether this mismatch produces a sufficiently
important **consumer decision problem** that existing mechanisms cannot solve.

In particular:

> Is there a meaningful decision that a consumer could make before
> installation/build if release-level implementation support were represented
> machine-readably?

This is the most important question in the research.

---

## 2. What is the root cause of the implementation restriction?

A statement such as:

```text id="fjh6yq"
CPython only
```

is not enough to determine what metadata mechanism is appropriate.

The restriction may arise from:

* runtime semantic dependence on CPython;
* build/toolchain limitations;
* alternative-interpreter bugs;
* ABI/configuration requirements;
* private implementation APIs;
* explicit maintainer support policy;
* conditional or fallback behavior;
* an implementation-specific dependency or component.

The current taxonomy is:

```text id="n7nj3q"
A — runtime semantic restriction
B — build/toolchain restriction
C — alternative-implementation bug/workaround
D — ABI/configuration restriction
E — private implementation usage
F — explicit support-policy declaration
G — conditional/fallback support
H — dependency/component restriction
```

The unresolved question is:

> Which of these categories actually produces the residual problem that
> existing packaging metadata cannot represent?

This is why `evidence/root-cause-taxonomy.md` precedes the final proposal.

---

## 3. Is "supported" the correct semantic concept?

A field such as:

```toml id="5x8x4b"
supported-implementation = ["cpython"]
```

could mean several different things:

* tested by CI;
* officially supported by maintainers;
* expected to work;
* known to work;
* supported for installation;
* supported for runtime execution;
* supported for building;
* supported for all configurations of that implementation.

These meanings are not equivalent.

For example:

```text id="w2t1qs"
known to work on CPython
        ≠
supported only on CPython
```

and:

```text id="cx6q7a"
not tested on PyPy
        ≠
known incompatible with PyPy
```

A standard must define the intended semantics precisely if the information is
to become machine-actionable.

---

## 4. Is this actually a requirement or a support declaration?

This is one of the central unresolved terminology questions.

Consider:

```text id="d9r8l2"
Requires-Implementation: cpython
```

This sounds like an installation requirement.

But a project may instead mean:

```text id="m2p4z7"
Supported-Implementation: cpython
```

which describes the producer's known/supportable implementation set.

These are materially different concepts.

A requirement can imply:

```text id="w4c7kq"
outside the set → candidate is invalid
```

while support metadata could mean:

```text id="a8n3ds"
inside the set → producer declares support
outside the set → support is not declared
```

The research currently considers **positive support semantics** more promising
than treating implementation identity as another hard requirement, but this is
not yet a recommendation.

---

## 5. Is the field normative or informational?

Possible models include:

### Informational

The field describes maintainer intent but does not affect installation.

### Advisory

Tools may warn when the current implementation is not listed.

### Candidate-selection metadata

Resolvers/installers may use the field to eliminate or deprioritize
candidates.

### Mandatory compatibility constraint

Installers must reject a candidate when the current implementation is not
listed.

The research currently does not select one of these.

The strongest unresolved question is:

> Can the metadata be useful for machine-assisted decisions without turning an
> imperfect maintainer support declaration into a hard compatibility constraint?

This is particularly important because implementation support declarations may
become stale.

---

## 6. What does absence mean?

Possible choices:

### A. Absence means all implementations are supported

This is attractive for simplicity but unsafe.

### B. Absence means unknown

This is semantically conservative but may reduce usefulness.

### C. Absence means no normative compatibility declaration

This preserves the existing meaning of distributions that predate the field.

The research currently prefers **C**.

Existing distributions must not suddenly become incompatible simply because they
do not contain a field that did not previously exist.

---

## 7. What does an empty list mean?

Possible meanings include:

```toml id="j8p3zq"
supported-implementation = []
```

could mean:

* no implementations supported;
* no information;
* invalid metadata.

An empty declaration could easily create ambiguity.

The current design hypothesis is therefore that an empty list should either be
prohibited or given a carefully defined meaning.

This remains unresolved.

---

## 8. How should partial implementation support be represented?

Implementation support may not be binary.

A project could support:

```text id="t6m0kw"
CPython
PyPy
```

but only for certain Python versions.

Or:

```text id="h5r2jc"
CPython
```

while excluding:

```text id="s9k1fv"
CPython free-threaded
```

or:

```text id="q7c4xe"
CPython debug builds
```

The implementation dimension therefore interacts with:

* Python version;
* ABI features;
* platform;
* architecture;
* build configuration.

A useful representation must not imply that implementation identity alone
fully describes compatibility.

---

## 9. Is implementation identity sufficient?

No.

A project can support:

```text id="n3s6px"
CPython
```

but reject:

```text id="e2j7kw"
CPython free-threaded
```

or:

```text id="p4d8mv"
CPython debug
```

PEP 780 is relevant to these ABI/configuration dimensions.

The proposed field should therefore not become a general-purpose interpreter
compatibility language.

The unresolved question is:

> What exact compatibility dimension would implementation-support metadata
> own, and what dimensions must remain owned by other mechanisms?

---

## 10. How should ABI metadata interact with implementation metadata?

A future candidate-selection process could conceptually evaluate:

```text id="u7v3de"
implementation identity
+
Python version
+
ABI features
+
platform
+
wheel tags
```

The standards need to define which mechanism owns each dimension.

For example:

```text id="z4x8pn"
Implementation identity
    → implementation-support metadata

Python version
    → Requires-Python

ABI features
    → PEP 780 / artifact metadata

Platform
    → platform/wheel metadata

Individual artifact compatibility
    → wheel tags / variants
```

This is currently a conceptual boundary, not a finalized architecture.

---

## 11. Runtime vs build-time compatibility

Consider two statements:

```text id="0v4t2x"
The package can only run on CPython.
```

and:

```text id="j9k5sw"
The package's build process must execute under CPython.
```

These are different claims.

The first is about runtime support.

The second may be about:

* build dependencies;
* host dependencies;
* toolchains;
* build environment;
* external libraries;
* alternative-interpreter build limitations.

They should not automatically use the same metadata semantics.

PEP 725 must therefore be considered before creating a second vocabulary for
build-environment requirements.

---

## 12. Could PEP 725 solve the relevant cases?

PEP 725 is relevant to external dependencies and build/host requirements.

The current research question is not simply:

> “Can PEP 725 describe something involving CPython?”

It is:

> “Does PEP 725 cover the actual residual problem demonstrated by the
> strongest implementation-support cases?”

If the root cause is a build dependency or host requirement, PEP 725 may be
the more appropriate mechanism.

If the root cause is a runtime support boundary, a separate mechanism may still
be relevant.

This distinction must be demonstrated case-by-case.

---

## 13. Could the alternative-interpreter problem belong outside packaging?

Some CPython-only behavior may exist because:

* an alternative interpreter lacks a feature;
* an alternative interpreter has a bug;
* compatibility layers are incomplete;
* a project is working around an interpreter defect.

In those cases, package metadata may not be the best long-term solution.

For example:

```text id="x1z7qh"
package fails on PyPy
        ↓
PyPy compatibility bug
```

could be better addressed by fixing PyPy rather than declaring the package
permanently incompatible with PyPy.

The research must therefore distinguish:

```text id="j7k3pd"
package support boundary
```

from:

```text id="p9m4ax"
temporary ecosystem compatibility gap
```

---

## 14. Could private implementation usage be intentionally outside metadata?

A package may deliberately depend on implementation-specific internals.

For example:

```text id="m3f6ws"
CPython private API
        ↓
project is intentionally CPython-specific
```

The producer may effectively be saying:

> “This package is for CPython; other implementations are outside its intended
> use.”

The unresolved question is whether such intentional private implementation
usage should be represented as a normative installation constraint.

The research should not assume that every unsupported environment requires
metadata.

---

## 15. Could Trove classifiers solve the problem?

Classifiers already allow projects to say:

```text id="z2q6vr"
Programming Language :: Python :: Implementation :: CPython
```

and:

```text id="v5j1kd"
Programming Language :: Python :: Implementation :: PyPy
```

This demonstrates that the ecosystem already has a vocabulary for describing
implementation targeting.

The unresolved question is whether classifiers are sufficient for the
important use cases.

Specifically:

> Is the missing property primarily **machine actionability and defined
> semantics**, rather than the absence of an implementation vocabulary?

If classifiers already provide enough information for testing, discovery, and
human decision-making, a new field may not be justified.

If consumers need a defined compatibility meaning, the distinction becomes
more significant.

---

## 16. Why would projects maintain a new field?

Adoption is an independent problem.

A new field introduces maintenance work:

```text id="b8v0cz"
project author
    ↓
declare support
    ↓
keep declaration current
    ↓
ensure it matches actual behavior
```

The research has already identified the concern that projects do not
consistently maintain implementation classifiers.

That raises a fundamental question:

> Why would a project maintain a new normative field if it does not already
> maintain descriptive implementation classifiers reliably?

A proposal needs a concrete consumer benefit strong enough to justify this
additional maintenance burden.

---

## 17. Could incorrect declarations cause more harm than they prevent?

A machine-actionable declaration creates a new failure mode.

For example:

```text id="y4r8nv"
Supported-Implementation: cpython
```

could become stale after the project gains PyPy support.

Conversely:

```text id="h2q6st"
Supported-Implementation: cpython, pypy
```

could falsely imply support that has not actually been tested.

Possible responses include:

* informational semantics;
* warning-only behavior;
* conservative candidate selection;
* validation tooling;
* CI verification;
* repository-side checks;
* mandatory installer rejection.

The research has not established which approach is appropriate.

---

## 18. Could failed-build caching solve the practical problem?

Caching failed builds provides an alternative for some performance-oriented
source-build scenarios:

```text id="q1r5vm"
first attempt
    ↓
build failure
    ↓
cache result
    ↓
avoid repeating equivalent work
```

This weakens the argument that a new field is automatically necessary merely
because source builds can fail.

However, caching is not semantically equivalent to support metadata.

A cache records:

```text id="v8m2cp"
observed result in environment X
```

while support metadata would represent:

```text id="d5k7nz"
producer declaration about release R
```

A cached failure:

* still requires a first attempt;
* may be environment-specific;
* may result from a temporary failure;
* may become stale;
* may not transfer between environments.

The unresolved question is therefore:

> How much of the practical value claimed for implementation metadata is about
> semantic compatibility, and how much is merely about avoiding repeated failed
> work?

---

## 19. Could wheel tags solve the problem?

For an already-built wheel, wheel tags are the established artifact-level
mechanism.

The difficult scenario is:

```text id="r8x1md"
sdist only
    ↓
installer must decide whether to build
```

The compatible wheel does not yet exist.

This is where a release-level support declaration could theoretically provide
information before a build is attempted.

However, the research must still determine whether:

* the restriction is actually encoded elsewhere;
* build metadata is the correct solution;
* the index can expose the information early enough;
* or the expected benefit is too small to justify new metadata.

Wheel tags and release-level support metadata should therefore be treated as
different layers rather than direct replacements.

---

## 20. Should PEP 825 variants be involved?

PEP 825 addresses wheel variants and index-level artifact compatibility.

This may help represent cases where compatibility depends on additional
artifact properties.

But variants remain fundamentally **artifact-level**.

The unresolved question is whether a release can have a support boundary that
cannot be cleanly represented by its individual artifact variants.

If so, a release-level declaration may still have a distinct role.

If not, variants may eliminate some of the apparent need for new metadata.

---

## 21. Does Core Metadata need to be extended?

Core Metadata is a plausible location for release-level compatibility data.

But adding a field has costs:

* specification complexity;
* build-backend support;
* metadata validation;
* installer behavior;
* repository behavior;
* documentation;
* backwards compatibility;
* maintenance of the implementation vocabulary;
* risk of stale producer declarations.

The benefit must justify those costs.

The research should therefore not begin with:

```text id="k4c9vz"
"We need a Core Metadata field."
```

but with:

```text id="x8d1qm"
"We have a consumer problem that existing mechanisms cannot adequately solve."
```

Only then should Core Metadata become the primary design candidate.

---

## 22. Could the information belong at the index/repository layer instead?

A release-level compatibility statement could potentially be exposed through
repository metadata rather than embedded directly in Core Metadata.

PEP 658 and related repository metadata mechanisms make release metadata
available separately from artifact downloads.

This raises a separate architectural question:

> Does the information need to be part of the distribution's canonical Core
> Metadata, or does the primary consumer need it at the repository/index layer?

The answer depends partly on whether the intended consumer is:

* an installer;
* a resolver;
* a package index;
* a test/fuzzing system;
* an ecosystem analysis tool;
* or another package consumer.

---

## 23. How would metadata be obtained?

PEP 658 and PEP 714 allow repositories to expose Core Metadata separately.

However, metadata sidecars are optional.

Therefore a design cannot assume that every package index will expose the field
before an artifact is downloaded.

Possible acquisition paths include:

```text id="e6v1pn"
index metadata
    ↓
Core Metadata
    ↓
artifact metadata
    ↓
source distribution
```

The research must determine whether the desired consumer decision can actually
be made at the point where the information becomes available.

---

## 24. Should metadata be release-level or artifact-level?

The research currently prefers **release-level semantics** for producer support.

A release can contain:

* an sdist;
* multiple wheels;
* wheels for different platforms;
* wheels for different ABIs;
* potentially different variants.

Wheel tags already describe individual artifact compatibility.

The proposed support declaration would instead describe the producer's
support claim for the release as a whole.

However, this creates an important question:

> Can one implementation-support declaration accurately describe all artifacts
> and source-build paths belonging to a release?

If not, the model may need more nuanced semantics.

---

## 25. Can implementation support be conditional?

A project may support an implementation only under particular conditions.

For example:

```text id="z5t9mc"
CPython:
    supported

PyPy:
    supported when optional dependency X is available

GraalPy:
    supported for pure-Python functionality only
```

This raises the possibility that implementation support is not always a simple
set.

The research must determine whether conditional support belongs:

* in implementation metadata;
* in dependency markers;
* in artifact metadata;
* in project documentation;
* or nowhere in normative installation metadata.

A field that attempts to encode arbitrary compatibility logic could become
unmanageably complex.

---

## 26. Is implementation vocabulary open-ended?

PEP 421 deliberately uses an implementation identity rather than a closed
registry.

A standard should avoid creating a second incompatible registry if possible.

The unresolved questions include:

* How are new implementations represented?
* Is the value an implementation name?
* Who defines canonical names?
* How are forks or derivatives represented?
* Can a producer use an implementation family?
* How should aliases be handled?

The vocabulary must remain compatible with the open-ended nature of Python
implementations.

---

## 27. What would an installer actually do with the information?

This question remains deliberately unresolved.

Possible behavior includes:

```text id="y7v3rc"
supported
    → normal candidate

not listed
    → deprioritize

not listed
    → warning

not listed
    → reject

unknown
    → normal behavior
```

The correct behavior depends on the semantics of the field.

A particularly important question is whether the resolver should distinguish:

```text id="q2j6hf"
producer says unsupported
```

from:

```text id="h5k8wm"
producer makes no declaration
```

The research currently favors preserving that distinction.

---

## 28. What would a non-installer consumer do?

The metadata could potentially be useful to:

* compatibility test runners;
* fuzzing infrastructure;
* package indexes;
* dependency analysis;
* package quality tooling;
* ecosystem research;
* automated CI matrix generation.

But each consumer may need different semantics.

For example:

```text id="r5n2cx"
testing tool:
    "not listed" → skip or mark unknown

installer:
    "not listed" → probably do nothing

hard compatibility checker:
    "not listed" → insufficient evidence
```

This is an argument against assuming that installer rejection is the only
possible consumer.

---

## 29. How common is the problem?

The current examples establish that the pattern exists.

They do not establish ecosystem-wide prevalence.

The research must not make claims such as:

```text id="g3p7zm"
"X% of PyPI packages have this problem"
```

without a reproducible release-level corpus and explicit methodology.

The important measurement questions are:

* How many releases explicitly declare implementation restrictions?
* How many publish implementation-generic wheels?
* How many also publish an sdist?
* How many restrictions are runtime rather than build/ABI/dependency issues?
* How many create an actual pre-install candidate-selection problem?
* How many would be solved by existing mechanisms?

---

## 30. What evidence would falsify the proposal?

The research should define failure conditions before recommending a PEP.

The proposal becomes substantially weaker if investigation shows that:

* most apparent implementation restrictions are actually ABI or build issues;
* PEP 725 adequately covers the important build cases;
* wheel tags or variants cover the important artifact cases;
* classifiers satisfy the meaningful consumer use cases;
* implementation restrictions are too rare to justify ecosystem complexity;
* producer declarations are too unreliable for machine actionability;
* consumers do not need the information before installation;
* or failed-build caching provides most of the practical benefit.

A credible research process must allow the conclusion:

> No new standard is necessary.

---

# 31. Does the problem justify a new standard?

This remains the central question.

The current evidence supports the narrower observation:

```text id="u4x8pq"
Some releases have implementation-specific support boundaries
that are not represented by their generic wheel tags or
Requires-Python.
```

The research has **not yet established**:

```text id="m6q1zt"
therefore a new Core Metadata field is necessary.
```

The remaining proof obligation is:

```text id="c7w2pn"
real release
    ↓
explicit support boundary
    ↓
root cause understood
    ↓
existing mechanisms insufficient
    ↓
meaningful pre-install consumer decision
    ↓
new machine-readable declaration provides material benefit
    ↓
benefit justifies ecosystem cost
```

Only if this chain survives adversarial investigation should a new normative
metadata mechanism be recommended.

---

# 32. Current design hypothesis

If the research ultimately establishes that a new mechanism is justified, the
current leading hypothesis is a positive support declaration such as:

```toml id="e8p4qy"
supported-implementation = ["cpython", "pypy"]
```

rather than a requirement-style field such as:

```toml id="v3k7hs"
requires-implementation = ["cpython"]
```

The reason is semantic:

```text id="s9m2fd"
Requires-Implementation
    sounds like
hard installation requirement

Supported-Implementation
    sounds like
producer-declared support boundary
```

However:

> `Supported-Implementation` is a research hypothesis, not the current
> recommendation.

The research must first establish that a normative release-level support
declaration is needed at all.

---

# 33. Current research priority

The highest-priority unresolved questions are now:

1. **Root cause:** Why are the strongest real-world releases
   implementation-specific?
2. **Residuality:** Which cases survive PEP 725, ABI, artifact, dependency,
   classifier, and alternative-interpreter explanations?
3. **Consumer:** Who needs the information before installation/build?
4. **Decision:** What concrete decision changes when the information is known?
5. **Semantics:** Is the required concept support, requirement, or something
   else?
6. **Trust:** Can producer declarations be maintained accurately enough for
   automation?
7. **Prevalence:** Is the problem common enough to justify standardization?
8. **Placement:** If justified, does the information belong in Core Metadata,
   repository metadata, or another layer?

Until these questions are answered, the project should remain in the research
phase rather than prematurely becoming a PEP proposal.
