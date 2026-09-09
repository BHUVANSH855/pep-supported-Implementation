# Evidence Methodology

## Purpose

This repository is intended to be useful to packaging maintainers and standards authors, not merely persuasive.

The methodology therefore separates:

1. observed facts;
2. root-cause classification;
3. compatibility interpretation;
4. existing-mechanism analysis;
5. consumer decision analysis;
6. proposed semantics.

A package enters the strong evidence set only when the relevant facts can be independently checked and the case survives adversarial attempts to explain it using existing mechanisms.

The core methodological rule is:

> **Observed restriction ≠ root cause ≠ metadata gap ≠ standardization requirement.**

A second rule follows:

> **A real compatibility problem is not automatically a Core Metadata problem.**

The research must establish not merely that a restriction exists, but that the restriction creates a useful machine-actionable distinction at a point where existing packaging mechanisms cannot adequately express or use it.

---

## Evidence hierarchy

### Level 1 — normative specifications

Highest-confidence sources:

* Python Packaging User Guide specifications;
* accepted/final PEPs;
* current draft PEPs when explicitly identified as drafts.

Relevant material includes:

* Core Metadata;
* PEP 421 / implementation identity;
* PEP 425 / compatibility tags;
* PEP 508 / dependency specifiers and environment markers;
* PEP 643 / Core Metadata 2.1;
* PEP 658 / metadata availability;
* PEP 714 / metadata-related packaging behavior;
* PEP 725 / external build and host requirements;
* PEP 780 / ABI/environment features;
* PEP 825 / wheel variants;
* other directly relevant packaging specifications.

Normative specifications establish what existing mechanisms mean.

They do **not** by themselves establish that a new mechanism is needed.

### Level 2 — published distribution metadata

Examples:

* PyPI release metadata;
* `Requires-Python`;
* classifiers;
* `Requires-Dist`;
* distribution filenames;
* wheel tags;
* presence/absence of sdist and wheels;
* release timestamps;
* metadata versions.

Published metadata is especially important because the proposed mechanism would operate at the release/distribution level.

Where possible, the exact release must be identified rather than relying on current project metadata.

### Level 3 — producer documentation

Examples:

* README;
* installation documentation;
* support matrix;
* release notes;
* project website;
* documented interpreter requirements.

Producer documentation is strong evidence of intended support policy.

It is not necessarily proof of technical incompatibility.

### Level 4 — source and runtime evidence

Examples:

```python
if sys.implementation.name != "cpython":
    ...
```

Other useful evidence includes:

* implementation-specific imports;
* private CPython API usage;
* ABI checks;
* build-system checks;
* conditional compilation;
* runtime guards;
* CI matrices;
* implementation-specific test failures.

Source evidence becomes particularly strong when it directly enforces the claimed restriction.

A source-level check is still not automatically evidence that the restriction should become normative package metadata.

### Level 5 — ecosystem discussion

Discussions, issue trackers, mailing lists, and similar sources provide evidence of:

* proposed use cases;
* consumer pain;
* maintainer concerns;
* alternative solutions;
* adoption concerns;
* semantic disagreements.

They are evidence of ecosystem arguments and practice, not normative specifications.

---

## Source provenance requirements

Every strong case should preserve enough information for an independent reviewer to reproduce the observation.

At minimum record:

```text
project
exact release/version
release date
source/repository revision where relevant
published metadata
artifact filenames
producer documentation
source evidence
relevant discussion
```

For source-level claims, prefer a release-specific commit or immutable source revision over an unversioned `main` branch.

For runtime claims, distinguish:

```text
observed locally
```

from:

```text
reported by producer
```

and:

```text
inferred from source
```

Do not silently convert one evidence type into another.

---

## Release-level identity

The primary unit of analysis is the **release**, not merely the project.

A project can change implementation support between releases.

For example:

```text
release R1
    ↓
CPython only

release R2
    ↓
CPython + PyPy

release R3
    ↓
support changes again
```

Therefore a project-level statement such as:

```text
Project X supports CPython
```

is insufficient for strong residual evidence unless it can be tied to the exact release being evaluated.

This also matters for stale metadata.

A declaration that was correct for release `R1` may be incorrect for `R2`.

---

## Root-cause classification

Every claimed implementation restriction should first be classified using `evidence/root-cause-taxonomy.md`.

At minimum, distinguish:

* A — runtime semantic restriction;
* B — build/toolchain restriction;
* C — alternative-implementation bug/workaround;
* D — ABI/configuration restriction;
* E — private implementation usage;
* F — explicit support-policy declaration;
* G — conditional/fallback support;
* H — dependency/component restriction.

Do not count these as equivalent evidence.

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

Likewise:

```text
CPython-only dependency
```

must not automatically become:

```text
top-level package is CPython-only
```

The actual role of the implementation-specific behavior must be established.

---

## Root cause versus support policy

A producer may explicitly state:

```text
CPython only
```

for several different reasons.

The statement could represent:

1. a hard technical restriction;
2. a security guarantee;
3. an intentionally unsupported but technically functional environment;
4. a temporary implementation limitation;
5. a testing/support-policy boundary;
6. an artifact/build limitation;
7. an undocumented assumption.

These cases have different implications for metadata.

The research therefore records both:

```text
technical cause
```

and:

```text producer support policy
```

when they differ.

An explicit support statement is evidence of producer intent.

It is not automatically evidence that all other implementations are technically incapable of running the release.

---

## What counts as a strong residual case?

A strong case should answer the following questions.

| Question                                                                     | Requirement                                         |
| ---------------------------------------------------------------------------- | --------------------------------------------------- |
| Is the exact release identified?                                             | **Required**                                        |
| Can the producer's support statement be independently checked?               | **Required where a support statement is claimed**   |
| Is the relevant behavior/restriction independently evidenced?                | **Required for technical residuals**                |
| Is the Python version range separately expressible?                          | **Must be analyzed**                                |
| Is the wheel implementation tag generic or otherwise insufficient?           | **Strongly preferred for artifact-level residuals** |
| Is an sdist available?                                                       | **Strongly preferred for build-related residuals**  |
| Can PEP 508 express the package's own restriction?                           | **Must be analyzed**                                |
| Is this actually an ABI issue?                                               | **Must be checked**                                 |
| Is this actually a build/host requirement?                                   | **Must be checked**                                 |
| Could PEP 725 or another build mechanism solve it?                           | **Must be checked**                                 |
| Is this actually an alternative-implementation bug/workaround?               | **Must be checked**                                 |
| Is private implementation usage the real reason?                             | **Must be checked**                                 |
| Could wheel tags or variants express the artifact restriction?               | **Must be checked**                                 |
| Could existing classifiers communicate the positive support fact?            | **Must be tested**                                  |
| Is there a fallback or conditional-support explanation?                      | **Must be checked**                                 |
| Does the restriction apply to the whole release or only a component/feature? | **Must be checked**                                 |
| Does metadata change a pre-install/pre-build decision?                       | **Required for a strong residual**                  |
| Is there a concrete consumer benefit?                                        | **Required for a strong residual**                  |
| Is the proposed benefit unavailable through an alternative mechanism?        | **Required**                                        |
| Can the case be reproduced or independently audited?                         | **Required**                                        |

The standard is deliberately higher than:

```text
CPython only + py3-none-any
```

A generic wheel plus an implementation-specific statement is a **candidate signal**, not a completed residual case.

---

## Evidence status categories

The repository should distinguish at least these statuses:

### Confirmed residual

Use only when:

* the restriction is established;
* the root cause is understood;
* relevant existing mechanisms have been evaluated;
* the information is missing or insufficient at the point where the consumer must make a decision;
* and a concrete incremental consumer benefit is demonstrated.

### Strong candidate

The case has unusually strong evidence, but one or more residuality questions remain unresolved.

Examples may include:

* HAX 0.3.0;
* simple-ctx-log 0.0.3.

### Policy-only candidate

The producer explicitly limits support, but technical incompatibility has not been established or the distinction may primarily represent a support/security policy.

RestrictedPython is currently important evidence of this kind.

### Control/boundary case

The case is useful because it prevents an overly broad conclusion.

Examples include:

* packages with implementation-specific code and fallbacks;
* generic Python tags around native components;
* ABI-sensitive packages where wheel tags may already solve the problem.

### Unresolved candidate

The observed restriction is credible, but its root cause or consumer consequence has not been sufficiently established.

Likepy currently belongs closer to this category than to confirmed residual evidence.

### Rejected/non-residual

The apparent implementation restriction is adequately explained by an existing mechanism, is only an inference, or does not create a meaningful consumer decision.

---

## What we explicitly do NOT infer

### No PyPy wheel

Does **not** imply:

```text
PyPy unsupported
```

A project may simply choose not to publish a PyPy-specific wheel while supporting source builds or a generic wheel.

### CPython classifier

Does not prove:

```text
all other implementations are technically incompatible
```

It proves, at most, that the project has declared CPython in its descriptive classification.

### Missing PyPy classifier

Does not prove:

```text
PyPy unsupported
```

The safer interpretation for this research is:

```text
absence = no normative support declaration
```

unless the relevant classifier specification establishes stronger semantics.

### `sys.implementation` check

Does not automatically mean the project is CPython-only.

The code may:

* select an optimization;
* work around an implementation bug;
* choose a fallback;
* expose a feature only on one implementation.

### Native extension

Does not automatically justify a new field.

Wheel tags and ABI mechanisms may already solve the artifact-selection problem.

### CPython-only dependency

Does not automatically make the parent package CPython-only.

The parent may:

* make the dependency optional;
* use an alternative implementation;
* conditionally install it;
* isolate it to a feature;
* or provide a fallback.

### Generic wheel

Does not automatically mean:

```text
all implementations supported
```

It establishes artifact compatibility at the level encoded by the wheel filename.

---

## Existing-mechanism elimination

A residual candidate must be tested against existing mechanisms before being used as evidence for a new one.

At minimum evaluate:

```text
Requires-Python
PEP 508 markers
wheel Python tags
wheel ABI tags
wheel platform tags
PEP 780 ABI/environment features
PEP 825 variants
PEP 725 build/host metadata
source-build policy
Trove classifiers
dependency structure
```

The purpose is not to prove that every existing mechanism is inadequate.

The purpose is to determine whether the observed restriction actually belongs to a semantic space that those mechanisms do not represent.

---

## Residual-case test

For each candidate release, evaluate:

```text
                         ┌── Requires-Python
                         │
                         ├── PEP 508 markers
                         │
                         ├── wheel Python/ABI/platform tags
                         │
Release candidate ───────┼── PEP 780 ABI/environment features
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
                      Whole release affected?
                                  │
                                  ▼
                    Existing mechanism sufficient?
                                  │
                           ┌──────┴──────┐
                           │             │
                          YES            NO
                           │             │
                    existing path    residual candidate
                                         │
                                         ▼
                              Consumer decision identified?
                                         │
                                         ▼
                              Incremental benefit demonstrated?
```

The answer must be based on the actual release, not a hypothetical package.

---

## Consumer-decision test

The central question is not merely:

> Would this information be useful?

It is:

> **What concrete decision could a consumer make earlier or better if this information were standardized?**

Candidate decisions include:

* reject a release before installation;
* avoid selecting an sdist;
* avoid attempting an expensive source build;
* choose another release;
* choose another artifact;
* construct a testing matrix;
* avoid executing known-incompatible packages;
* provide a clearer diagnostic;
* prioritize packages known to support an implementation.

A strong residual should identify the decision explicitly.

For example:

```text
current path
    ↓
candidate appears compatible
    ↓
source build attempted
    ↓
implementation-specific failure
```

versus:

```text
support declaration available
    ↓
candidate recognized as unsuitable
    ↓
alternative candidate selected
```

If there is no meaningful decision at the point where the metadata would be consumed, the case is weak evidence for normative metadata.

---

## Pre-install versus post-install evidence

The research must distinguish between information useful:

### Before candidate selection

Examples:

* selecting among releases;
* selecting among wheels;
* avoiding incompatible distributions.

### Before source build

Examples:

* avoiding a known incompatible sdist;
* avoiding an expensive build attempt.

### During installation

Examples:

* producing a better diagnostic;
* selecting a conditional dependency.

### After installation

Examples:

* runtime guard;
* telemetry;
* test results;
* cached failures.

A runtime failure is evidence that incompatibility exists.

It does not automatically demonstrate that a resolver could or should have prevented it earlier.

---

## Candidate-selection evidence

Do not claim that an installer would currently select an incompatible release unless this has been demonstrated or established by the relevant specification.

For example:

```text
generic wheel exists
```

is not sufficient to claim:

```text
pip will install it on PyPy
```

unless the actual candidate-selection behavior has been tested or otherwise established.

Likewise:

```text
classifier says CPython
```

is not sufficient to claim:

```text
resolver ignores the classifier
```

without understanding what the resolver is specified or observed to do.

Controlled experiments should record:

* command;
* resolver/tool version;
* target environment;
* available artifacts;
* result;
* limitations of the experiment.

---

## Classifier experiment

The classifier question is a first-class research track.

Test at least these cases.

### A — Positive support

```text
CPython classifier
+
documentation/CI says CPython supported
```

Question:

> Can the classifier safely communicate positive support without changing its established semantics?

### B — Missing implementation classifier

```text
CPython classifier
no PyPy classifier
```

Question:

> Does absence mean unsupported, or merely undeclared?

The repository currently uses the safer interpretation:

```text
absence = no normative declaration
```

unless stronger evidence establishes otherwise.

### C — Classifier disagreement

Find releases where:

```text
classifier says CPython
documentation/CI says PyPy supported
```

or the reverse.

These are high-value cases because they directly test whether classifiers can be made machine-actionable without changing historical metadata semantics.

### D — No classifier

Find releases with explicit implementation-support statements but no implementation classifier.

This measures whether the existing classifier vocabulary is actually being used for the relevant information.

### E — Support transitions

Track releases where implementation support changes over time.

This tests whether positive support declarations can remain accurate and maintainable.

---

## Positive support versus exclusion

The research must keep two possible semantic models separate.

### Exclusion model

```text
implementation X
    ↓
release must not be selected
```

This behaves like a requirement.

### Positive-support model

```text
implementation X
    ↓
producer explicitly supports this release
```

This does not necessarily mean:

```text
all omitted implementations are unsupported
```

These models have different consequences for:

* resolver behavior;
* omission semantics;
* stale metadata;
* ecosystem adoption;
* producer burden;
* false negatives;
* compatibility claims.

The research must not choose between them merely because one produces a convenient syntax.

---

## Support versus requirement

The methodology treats the following as distinct properties:

```text
Requires-Python
    = required interpreter version range

implementation requirement
    = environment must satisfy implementation constraint

supported implementation
    = producer declares support for an implementation

observed compatibility
    = software was observed to work

support policy
    = producer's intended compatibility boundary
```

A future metadata field must identify which of these it represents.

If the field is a requirement, omission and rejection semantics become central.

If the field is a positive support declaration, omission must not silently become universal exclusion.

---

## Conditional and fallback support

The corpus deliberately includes projects that:

* support CPython and PyPy;
* inspect `sys.implementation`;
* conditionally build native extensions;
* have CPython-only optional dependencies;
* publish fewer wheels than they test;
* have platform-specific but implementation-generic wheels;
* use CPython-specific code while retaining a fallback.

These controls are essential because otherwise an automated scan would overestimate the problem.

The methodology therefore asks:

```text
implementation-specific behavior
        ↓
hard release-level restriction?
        ↓
or conditional/fallback behavior?
```

Only the former is strong evidence for a general compatibility declaration.

---

## Dependency propagation

Implementation support must not be inferred transitively without examining dependency semantics.

For example:

```text
A
|
+-- B
     |
     +-- CPython-specific component
```

does not establish:

```text
A = CPython-only
```

because A may:

* select B conditionally;
* provide another dependency;
* use B only for an optional feature;
* provide a fallback;
* isolate the incompatible component.

The methodology therefore treats top-level support as a property of the release's effective behavior, not a simple transitive property of its dependency graph.

---

## ABI and artifact controls

A proposed implementation-support field must not duplicate information already represented by artifact compatibility.

A native extension may be restricted by:

```text
Python ABI
platform
architecture
free-threading configuration
other environment features
```

These dimensions may already be represented through wheel tags or related specifications.

Therefore:

```text
native extension
        ≠
new implementation metadata required
```

The case must first establish that the producer-level implementation distinction survives artifact-level analysis.

---

## Build and host controls

Source builds require separate analysis of:

```text
build dependencies
host dependencies
compiler/toolchain
external libraries
environment features
implementation identity
```

PEP 725 is particularly relevant when the actual problem concerns external build or host requirements.

A source build that fails on an alternative implementation is therefore not automatically a residual implementation-support case.

The methodology asks:

> Is the failure caused by the implementation itself, or by a build/host prerequisite that belongs in another metadata layer?

---

## Alternative-interpreter bug controls

An observed failure on PyPy, GraalPy, or another implementation may represent a defect in that implementation rather than a permanent property of the package.

Therefore distinguish:

```text
package requires CPython
```

from:

```text
package works elsewhere in principle,
but implementation X currently has bug Y
```

A temporary workaround or implementation bug is important evidence for ecosystem compatibility, but does not necessarily justify a producer-authored permanent exclusion.

---

## Failed-build caching as a competing solution

Build-failure caching is a serious alternative when the proposed benefit is primarily:

```text
avoid repeating expensive failed work
```

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

However, caching is not semantically equivalent to producer metadata.

Caching records:

```text
observed result in environment X
```

whereas support metadata would represent:

```text
producer declaration about release R
```

The comparison must therefore ask two separate questions:

1. Can caching solve the performance problem?
2. Does standardized producer metadata provide information that caching cannot provide before the first attempt?

A new field should not be justified merely because caching is imperfect.

---

## Consumer-benefit test

A metadata proposal should not be justified only by:

```text
"the information would be nice to know."
```

For each strong case ask:

```text
current resolver/tool path
        ↓
candidate selected
        ↓
expensive work?
        ↓
failure / wasted work / uncertainty
```

versus:

```text
support metadata available
        ↓
candidate classified earlier
        ↓
better selection / testing / diagnostic
```

Then identify the concrete benefit:

* build time avoided;
* network/download work avoided;
* repeated failure avoided;
* diagnostic improvement;
* resolver quality improvement;
* testing/fuzzing matrix improvement;
* safer environment selection.

Where possible, measure the benefit rather than merely asserting it.

---

## Quantitative claims

The current repository does **not** claim a PyPI-wide prevalence percentage.

The current corpus is intentionally enriched for implementation-related cases.

Therefore:

```text
6/32 packages are CPython-only
```

must not be presented as:

```text
18.75% of PyPI is CPython-only
```

A real prevalence estimate would require:

1. a reproducible corpus;
2. a defined sampling methodology;
3. release-level rather than project-level counting where appropriate;
4. explicit classification criteria;
5. treatment of missing and ambiguous data;
6. independent verification of sampled cases.

Ideally this would use PyPI distribution metadata or another public dataset.

---

## Evidence ledger

Every strong case should eventually have:

```text
project
version
release date
Requires-Python
classifiers
Requires-Dist
sdist
wheel filenames
wheel tags
implementation statement
root-cause classification
source/runtime evidence
dependency markers
ABI evidence
build/host evidence
alternative-interpreter evidence
candidate-selection effect
consumer decision
consumer benefit
alternative solution analysis
source URLs
confidence
```

Where an item is unavailable, record:

```text
unknown
```

rather than infer a value.

This makes the repository auditable by someone who disagrees with the proposal.

---

## Confidence model

Confidence should be attached to individual claims rather than to an entire package.

For example:

```text
producer says CPython only
    → high confidence

runtime guard exists
    → high confidence

PyPy definitely fails
    → lower confidence unless reproduced

resolver would select release on PyPy
    → requires resolver evidence

new metadata would prevent the failure
    → requires consumer-path analysis
```

This prevents strong evidence at one layer from being accidentally propagated into an unsupported conclusion at another layer.

---

## Reproducibility target

A future automated study should ideally use a release-level dataset such as PyPI distribution metadata and classify:

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
release history
```

The automated phase should be treated as a **candidate-generation system**, not as the final classifier.

For example:

```text
CPython classifier
+
generic wheel
+
CPython-only text
```

should produce:

```text
candidate
```

rather than:

```text
confirmed residual
```

The final classification requires source/documentation/root-cause review.

---

## Adversarial validation procedure

The strongest cases should undergo the following sequence:

```text
1. Identify exact release
        ↓
2. Preserve published metadata
        ↓
3. Verify producer support statement
        ↓
4. Verify source/runtime behavior
        ↓
5. Classify root cause
        ↓
6. Determine whether whole release is affected
        ↓
7. Test Requires-Python
        ↓
8. Test PEP 508 applicability
        ↓
9. Test wheel/artifact compatibility
        ↓
10. Test ABI/configuration explanations
        ↓
11. Test build/host explanations
        ↓
12. Check alternative-interpreter bugs/workarounds
        ↓
13. Check private-API explanations
        ↓
14. Check classifiers
        ↓
15. Identify actual consumer decision
        ↓
16. Establish when the decision must occur
        ↓
17. Compare alternative solutions
        ↓
18. Measure incremental benefit
        ↓
19. Assess producer-maintenance/trust implications
        ↓
20. Decide whether the case is truly residual
```

This procedure is deliberately capable of producing a **negative result**.

If existing mechanisms explain the case adequately, the case should be removed from the residual set even if the original observation was genuine.

---

## Avoiding confirmation bias

The research must actively search for evidence against the proposal.

This includes:

* packages that support multiple implementations despite implementation-specific code;
* packages where wheel tags already solve the restriction;
* packages where `Requires-Python` is sufficient;
* packages where PEP 508 markers solve dependency selection;
* packages where PEP 725 is the appropriate layer;
* packages where a runtime failure is actually an alternative-interpreter bug;
* packages where producer support is merely a testing policy;
* packages where classifiers already communicate the intended information;
* packages where adding a new field would be stale or costly to maintain;
* alternative tooling strategies such as failed-build caching.

A successful research outcome may therefore be:

```text
new metadata justified
```

or:

```text
existing mechanisms sufficient
```

or:

```text
problem real, but belongs outside Core Metadata
```

The methodology does not assume the first outcome.

---

## Evidence ledger interpretation

The repository should distinguish at least four conclusions:

```text
Observation established
```

means the behavior or declaration is real.

```text
Residual established
```

means existing mechanisms do not adequately represent or use the relevant information for the identified consumer decision.

```text
Standardization benefit established
```

means a new mechanism provides material incremental value.

```text
PEP design justified
```

means the accumulated evidence is sufficient to begin specifying normative semantics.

These are separate milestones.

A large collection of observations does not automatically establish the latter conclusions.

---

## Current research gate

The current project should not move from empirical research to PEP design merely because several CPython-only projects have been found.

The research gate is:

```text
real-world cases
        ↓
reproducible release evidence
        ↓
root causes
        ↓
existing mechanisms eliminated
        ↓
residual cases
        ↓
actual consumer decisions
        ↓
incremental benefit
        ↓
alternative solutions compared
        ↓
semantic model selected
        ↓
PEP design
```

The semantic model should only be selected after the evidence establishes what kind of information is actually missing.

In particular, the research must remain open between:

```text
Requires-Implementation
```

and:

```text
Supported-Implementation
```

as well as:

```text
existing mechanisms / no new field
```

until the residual corpus provides sufficient evidence.

---

## Current methodological conclusion

The current methodology establishes a deliberately conservative standard:

> A package that claims CPython-only support is evidence of an implementation-support boundary. It is not, by itself, evidence of a metadata gap.

The strongest evidence requires a complete chain:

```text
exact release
    ↓
verified restriction
    ↓
understood root cause
    ↓
existing mechanisms insufficient
    ↓
specific consumer decision
    ↓
actionable before the relevant cost/failure
    ↓
measurable incremental benefit
    ↓
credible alternative mechanisms considered
```

Only after that chain is established should the research conclude that standardized implementation-support metadata is justified.

The repository therefore remains willing to reach any of the following outcomes:

```text
A. New Core Metadata field justified

B. Existing metadata needs clearer semantics
   but no new field is required

C. The problem belongs in artifact/build/index metadata

D. Tooling improvements are sufficient

E. The observed problem is real but too uncommon,
   unstable, or ambiguous to justify standardization
```

The purpose of the methodology is not to prove the proposal.

It is to make the eventual conclusion defensible even if the conclusion is that the proposal should **not** be adopted.
