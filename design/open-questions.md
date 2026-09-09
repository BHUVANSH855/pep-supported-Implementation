# Open Questions

This document deliberately records questions that the research has not resolved.

The questions are ordered from the underlying problem to the possible metadata design. This is intentional: the research should establish that a standardization problem survives existing mechanisms **before** optimizing the syntax or semantics of a proposed field.

The central distinction throughout this document is:

```text
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

The research has identified real releases where a producer explicitly limits support to CPython while publishing artifacts whose compatibility metadata is more generic than that support boundary.

Examples now include cases such as:

```text
producer:
    CPython only

artifact:
    py3-none-any

sdist:
    present

Requires-Python:
    Python-version restriction only

classifier:
    implementation information, but not a normative compatibility constraint
```

The strongest evidence includes runtime guards and explicit support statements. For example, HAX 0.3.0 contains an explicit CPython-only runtime check, while simple-ctx-log documents CPython-only behavior associated with `sys._getframe`.

This establishes that the pattern exists.

It does **not** establish that the pattern creates a sufficiently important packaging problem.

The unresolved question is:

> Is there a meaningful consumer decision that could be made before installation or source build if release-level implementation support were represented machine-readably?

In particular:

* Would a resolver choose a different release?
* Would an installer avoid an otherwise apparently compatible sdist?
* Would a build system avoid an expected failure?
* Would a compatibility-testing system gain materially better information?
* Would ecosystem analysis become substantially more accurate?

This remains the most important question in the research.

---

## 2. What is the root cause of the implementation restriction?

A statement such as:

```text
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

The unresolved question is:

> Which categories actually produce a residual packaging problem that existing metadata cannot represent?

A release should not be counted merely because its documentation says "CPython only."

The research must establish why the restriction exists and whether that reason belongs in packaging metadata at all.

---

## 3. Is "supported" the correct semantic concept?

A field such as:

```toml
supported-implementation = ["cpython"]
```

could mean:

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

```text
known to work on CPython
        ≠
supported only on CPython
```

and:

```text
not tested on PyPy
        ≠
known incompatible with PyPy
```

The research now includes real examples where explicit support policy is stronger than mere lack of testing, but that still does not define a universal semantic rule.

The unresolved question is:

> What exactly would a producer be asserting by declaring an implementation "supported"?

A standard must define this precisely before the information can safely become machine-actionable.

---

## 4. Is this actually a requirement or a support declaration?

This remains one of the central unresolved terminology questions.

Consider:

```text
Requires-Implementation: cpython
```

This sounds like an installation requirement.

A project may instead mean:

```text
Supported-Implementation: cpython
```

which describes the producer's declared support boundary.

These are materially different concepts.

A requirement can imply:

```text
outside the set → candidate is invalid
```

while support metadata could mean:

```text
inside the set → producer declares support

outside the set → producer does not declare support
```

The research currently considers **positive support semantics** more promising than treating implementation identity as another hard requirement, but this is not a recommendation.

The unresolved question is whether the consumer problem actually requires either concept.

---

## 5. Is the field normative or informational?

Possible models include:

### Informational

The field describes maintainer intent but does not affect installation.

### Advisory

Tools may warn when the current implementation is not listed.

### Candidate-selection metadata

Resolvers or installers may use the field to eliminate or deprioritize candidates.

### Mandatory compatibility constraint

Installers must reject a candidate when the current implementation is not listed.

The research currently does not select one of these.

A particularly important question is:

> Can implementation-support metadata provide useful machine-assisted information without turning an imperfect producer support declaration into a hard compatibility constraint?

The controlled classifier experiment is relevant here.

A package carrying a CPython implementation classifier but otherwise generic `py3-none-any` artifact was accepted by the tested pip path, demonstrating that the existing classifier is not functioning as a generic normative candidate restriction.

That establishes a distinction between **having implementation information** and **having machine-actionable compatibility semantics**.

It does not establish that the latter requires a new Core Metadata field.

---

## 6. What does absence mean?

Possible choices:

### A. Absence means all implementations are supported

This is attractive for simplicity but unsafe.

### B. Absence means unknown

This is semantically conservative but may reduce usefulness.

### C. Absence means no normative compatibility declaration

This preserves the existing meaning of distributions that predate the field.

The research currently favors **C** as the least disruptive interpretation if a field were eventually standardized.

Existing distributions must not suddenly become incompatible simply because they do not contain a field that did not previously exist.

However, even this assumption requires validation against the intended consumer semantics.

---

## 7. What does an empty list mean?

Possible meanings include:

```toml
supported-implementation = []
```

which could mean:

* no implementations supported;
* no information;
* invalid metadata.

An empty declaration could easily create ambiguity.

The current design hypothesis is therefore that an empty list should either be prohibited or given a carefully defined meaning.

This remains unresolved.

---

## 8. How should partial implementation support be represented?

Implementation support may not be binary.

A project could support:

```text
CPython
PyPy
```

but only for certain Python versions.

Or:

```text
CPython
```

while excluding:

```text
CPython free-threaded
```

or:

```text
CPython debug builds
```

The implementation dimension therefore interacts with:

* Python version;
* ABI features;
* platform;
* architecture;
* build configuration.

A useful representation must not imply that implementation identity alone fully describes compatibility.

The unresolved question is:

> How much implementation detail belongs in implementation-support metadata, and where should version, ABI, platform, and configuration restrictions remain represented?

---

## 9. Is implementation identity sufficient?

No single implementation name can describe every compatibility dimension.

A project can support:

```text
CPython
```

but reject:

```text
CPython free-threaded
```

or:

```text
CPython debug
```

PEP 780 is relevant to ABI/configuration dimensions.

The proposed concept should therefore not become a general-purpose interpreter compatibility language.

The unresolved question is:

> What exact compatibility dimension would implementation-support metadata own, and what dimensions must remain owned by other mechanisms?

This boundary is essential to prevent duplication with:

* `Requires-Python`;
* wheel tags;
* variant metadata;
* ABI/environment metadata;
* dependency markers;
* build requirements.

---

## 10. How should ABI metadata interact with implementation metadata?

A future candidate-selection process could conceptually evaluate:

```text
implementation identity
+
Python version
+
ABI features
+
platform
+
artifact compatibility
```

The standards need to define which mechanism owns each dimension.

A possible conceptual boundary is:

```text
Implementation identity
    → implementation-support metadata

Python version
    → Requires-Python

ABI/environment features
    → PEP 780 / artifact metadata

Platform
    → platform and wheel metadata

Individual artifact compatibility
    → wheel tags / variants
```

This is only a research boundary, not a finalized architecture.

The unresolved question is whether the implementation dimension is sufficiently independent to justify another normative mechanism.

---

## 11. Runtime vs build-time compatibility

Consider:

```text
The package can only run on CPython.
```

and:

```text
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

PEP 725 must therefore be considered before creating a second vocabulary for build-environment requirements.

The unresolved question is:

> Which observed implementation-specific failures are genuinely release runtime-support boundaries, rather than build-environment requirements?

---

## 12. Could PEP 725 solve the relevant cases?

PEP 725 is relevant to external dependencies and build/host requirements.

The question is not:

> Can PEP 725 describe something involving CPython?

The actual question is:

> Does PEP 725 cover the actual residual problem demonstrated by the strongest implementation-support cases?

If the root cause is a build dependency or host requirement, PEP 725 may be the more appropriate mechanism.

If the root cause is a runtime support boundary, a separate mechanism may still be relevant.

Every claimed residual case should therefore be evaluated against PEP 725 explicitly rather than merely mentioning it as related prior art.

---

## 13. Could the alternative-interpreter problem belong outside packaging?

Some CPython-only behavior may exist because:

* an alternative interpreter lacks a feature;
* an alternative interpreter has a bug;
* compatibility layers are incomplete;
* a project is working around an interpreter defect.

In those cases, package metadata may not be the best long-term solution.

For example:

```text
package fails on PyPy
        ↓
PyPy compatibility bug
```

could be better addressed by fixing PyPy rather than declaring the package permanently incompatible with PyPy.

The research must therefore distinguish:

```text
package support boundary
```

from:

```text
temporary ecosystem compatibility gap
```

The unresolved question is:

> How much evidence is required before a compatibility failure can legitimately be treated as a producer-declared support boundary?

---

## 14. Could private implementation usage be intentionally outside metadata?

A package may deliberately depend on implementation-specific internals.

For example:

```text
CPython private API
        ↓
project is intentionally CPython-specific
```

The producer may effectively be saying:

> This package is for CPython; other implementations are outside its intended use.

The unresolved question is whether intentional private implementation usage should become a normative installation constraint.

The simple-ctx-log evidence is relevant because it combines an explicit CPython-only description with use of `sys._getframe`.

However, the presence of a private API does not automatically establish that a new metadata field is necessary.

The research should ask:

* Is the private API essential?
* Is there a portable alternative?
* Is the package intentionally implementation-specific?
* Would the project actually want installers to reject alternative implementations?
* Could existing classifiers or documentation adequately communicate the policy?

---

## 15. Could explicit security/support policy justify different treatment?

Some projects are CPython-only for reasons stronger than ordinary compatibility.

RestrictedPython is an important example because its documentation and source distinguish CPython support from use on other implementations in the context of its security guarantees.

This raises a separate question:

> Is a producer's explicit safety or security support boundary materially different from ordinary implementation compatibility?

If a project says that use on another implementation could undermine a security property, informational metadata may be insufficient.

But this does not automatically imply that the correct solution is an implementation requirement.

The research must determine whether security-sensitive support boundaries require:

* a hard constraint;
* an advisory declaration;
* a separate security metadata mechanism;
* or no packaging-level mechanism.

---

## 16. Could Trove classifiers solve the problem?

Classifiers already allow projects to say:

```text
Programming Language :: Python :: Implementation :: CPython
```

and:

```text
Programming Language :: Python :: Implementation :: PyPy
```

This demonstrates that the ecosystem already has a vocabulary for implementation targeting.

The unresolved question is whether classifiers are sufficient for the important use cases.

The controlled experiment provides useful evidence:

```text
CPython classifier
+
Requires-Python >= 3.8
+
py3-none-any wheel
```

was still selected by the tested pip resolver path under an explicitly simulated PyPy target.

The tested uv path similarly resolved the generic artifact, although the tested interface did not provide an explicit PyPy implementation override, so this must not be interpreted as a complete PyPy-specific uv experiment.

The evidence therefore supports a narrower statement:

> Existing implementation classifiers are not presently equivalent to a normative implementation compatibility constraint in the tested packaging paths.

It does **not** establish:

> Therefore a new Core Metadata field is necessary.

The unresolved question remains whether the missing property is primarily:

```text
machine actionability + defined semantics
```

rather than:

```text
absence of an implementation vocabulary
```

---

## 17. Why would projects maintain a new field?

Adoption is an independent problem.

A new field introduces maintenance work:

```text
project author
    ↓
declare support
    ↓
keep declaration current
    ↓
ensure it matches actual behavior
```

The research has identified projects with explicit implementation classifiers and projects with explicit CPython-only documentation, but classifier presence is not sufficient evidence that a normative declaration would be maintained accurately.

This raises a fundamental question:

> Why would a project maintain a new normative field if it does not already maintain implementation classifiers reliably?

A proposal needs a concrete consumer benefit strong enough to justify this additional maintenance burden.

The burden becomes greater if the field is used for hard candidate rejection.

---

## 18. Could incorrect declarations cause more harm than they prevent?

A machine-actionable declaration creates new failure modes.

For example:

```text
Supported-Implementation: cpython
```

could become stale after the project gains PyPy support.

Conversely:

```text
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

The key unresolved question is:

> Is producer-declared implementation support sufficiently stable and trustworthy to justify automated candidate decisions?

This question should be answered empirically where possible rather than assumed.

---

## 19. Could failed-build caching solve the practical problem?

Caching failed builds provides an alternative for some performance-oriented source-build scenarios:

```text
first attempt
    ↓
build failure
    ↓
cache result
    ↓
avoid repeating equivalent work
```

This weakens the argument that a new field is automatically necessary merely because source builds can fail.

However, caching is not semantically equivalent to support metadata.

A cache records:

```text
observed result in environment X
```

while support metadata would represent:

```text
producer declaration about release R
```

A cached failure:

* still requires a first attempt;
* may be environment-specific;
* may result from a temporary failure;
* may become stale;
* may not transfer between environments.

The unresolved question is:

> How much of the practical value claimed for implementation metadata is about semantic compatibility, and how much is merely about avoiding repeated failed work?

If caching solves the primary practical problem, the justification for a new metadata field becomes weaker.

---

## 20. Could wheel tags solve the problem?

For an already-built wheel, wheel tags are the established artifact-level mechanism.

The difficult scenario is:

```text
sdist only
    ↓
installer must decide whether to build
```

The compatible wheel does not yet exist.

This is where a release-level support declaration could theoretically provide information before a build is attempted.

However, the research must still determine whether:

* the restriction is already encoded elsewhere;
* the artifact should instead use a more specific tag;
* the restriction is really a build requirement;
* the source distribution should be treated differently;
* the index could expose another form of compatibility information;
* or the expected benefit is too small to justify new metadata.

Wheel tags and release-level support metadata should therefore be treated as different layers rather than direct replacements.

---

## 21. Should PEP 825 variants be involved?

PEP 825 addresses wheel variants and index-level artifact compatibility.

This may help represent cases where compatibility depends on additional artifact properties.

But variants remain fundamentally artifact-level.

The unresolved question is:

> Does a release have a producer support boundary that cannot be cleanly represented by its individual artifact variants?

If every important case can be expressed through artifact selection, variants may eliminate some of the apparent need for a release-level declaration.

If an sdist remains inherently ambiguous despite complete artifact metadata, a separate release-level concept may still have a role.

---

## 22. Does Core Metadata need to be extended?

Core Metadata is a plausible location for release-level compatibility data.

But adding a field has costs:

* specification complexity;
* build-backend support;
* metadata validation;
* installer behavior;
* repository behavior;
* documentation;
* backwards compatibility;
* implementation-vocabulary maintenance;
* stale producer declarations;
* resolver complexity.

The benefit must justify those costs.

The research should therefore not begin with:

```text
"We need a Core Metadata field."
```

but with:

```text
"We have a consumer problem that existing mechanisms cannot adequately solve."
```

Only then should Core Metadata become the primary design candidate.

---

## 23. Could the information belong at the index/repository layer instead?

A release-level compatibility statement could potentially be exposed through repository metadata rather than embedded directly in Core Metadata.

PEP 658 and related repository metadata mechanisms make release metadata available separately from artifact downloads.

This raises a separate architectural question:

> Does the information need to be part of the distribution's canonical Core Metadata, or does the primary consumer need it at the repository/index layer?

The answer depends partly on whether the intended consumer is:

* an installer;
* a resolver;
* a package index;
* a test/fuzzing system;
* an ecosystem analysis tool;
* or another package consumer.

The information's location should follow the actual consumer workflow rather than being chosen in advance.

---

## 24. How would metadata be obtained?

PEP 658 and PEP 714 allow repositories to expose Core Metadata separately.

However, metadata sidecars are not guaranteed to be available for every distribution or repository.

Possible acquisition paths include:

```text
index metadata
    ↓
Core Metadata
    ↓
artifact metadata
    ↓
source distribution
```

The research must determine whether the desired consumer decision can actually be made at the point where the information becomes available.

A theoretically useful field that cannot reliably be obtained before the expensive operation it is intended to avoid may have limited practical value.

---

## 25. Should metadata be release-level or artifact-level?

The research currently treats release-level semantics as the main hypothesis for producer support.

A release can contain:

* an sdist;
* multiple wheels;
* wheels for different platforms;
* wheels for different ABIs;
* potentially multiple variants.

Wheel tags already describe individual artifact compatibility.

A support declaration would instead describe the producer's declared support for the release.

However, this creates an important question:

> Can one implementation-support declaration accurately describe all artifacts and source-build paths belonging to a release?

A release may contain:

```text
generic pure-Python wheel
+
CPython-specific native wheel
+
sdist
```

Those artifacts can have materially different compatibility properties.

If the release-level declaration cannot represent this without contradiction, the proposed abstraction may be wrong.

---

## 26. Can implementation support be conditional?

A project may support an implementation only under particular conditions.

For example:

```text
CPython:
    supported

PyPy:
    supported when optional dependency X is available

GraalPy:
    supported for pure-Python functionality only
```

This raises the possibility that implementation support is not always a simple set.

The research must determine whether conditional support belongs:

* in implementation metadata;
* in dependency markers;
* in artifact metadata;
* in project documentation;
* in another standardized mechanism;
* or nowhere in normative installation metadata.

A field that attempts to encode arbitrary compatibility logic could become unmanageably complex.

---

## 27. Is implementation vocabulary open-ended?

PEP 421 uses an implementation identity rather than requiring a closed registry.

A standard should avoid creating a second incompatible registry if possible.

The unresolved questions include:

* How are new implementations represented?
* Is the value an implementation name?
* Who defines canonical names?
* How are forks or derivatives represented?
* Can a producer use an implementation family?
* How should aliases be handled?
* What happens when an implementation changes identity?
* How should implementation versioning interact with Python versioning?

The vocabulary must remain compatible with the open-ended nature of Python implementations.

The existence of implementation classifiers also raises another question:

> Should any new metadata mechanism reuse the ecosystem's existing implementation identifiers rather than defining a new namespace?

---

## 28. What would an installer actually do with the information?

This question remains deliberately unresolved.

Possible behavior includes:

```text
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

```text
producer explicitly says unsupported
```

from:

```text
producer makes no declaration
```

The research currently favors preserving that distinction.

But even if that distinction is useful, it does not automatically imply that candidate rejection is appropriate.

---

## 29. What would a non-installer consumer do?

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

```text
testing tool:
    "not listed" → skip or mark unknown

installer:
    "not listed" → probably do nothing

hard compatibility checker:
    "not listed" → insufficient evidence
```

This is an argument against assuming that installer rejection is the only possible consumer.

The unresolved question is:

> Which consumer has the strongest concrete need, and can that need justify standardization independently of speculative future consumers?

---

## 30. Who is the actual consumer and what decision changes?

This question deserves separate treatment because it is the core proof obligation.

For every proposed use case, the research should identify:

```text
consumer
    ↓
information available today
    ↓
decision made today
    ↓
failure/cost of that decision
    ↓
new information
    ↓
different decision
    ↓
measurable benefit
```

For example, a claim such as:

```text
"pip should avoid trying this package on PyPy"
```

is incomplete unless the research establishes:

* why pip cannot know this today;
* why existing metadata is insufficient;
* whether the release would otherwise be selected;
* what operation would be avoided;
* how frequently this occurs;
* whether the producer declaration would be trustworthy.

The same discipline should apply to fuzzing, CI, package indexes, and ecosystem-analysis use cases.

---

## 31. How common is the problem?

The current examples establish that the pattern exists.

They do not establish ecosystem-wide prevalence.

The research must not make claims such as:

```text
"X% of PyPI packages have this problem"
```

without a reproducible release-level corpus and explicit methodology.

The important measurement questions are:

* How many releases explicitly declare implementation restrictions?
* How many contain implementation-specific runtime guards?
* How many publish implementation-generic wheels?
* How many also publish an sdist?
* How many restrictions are runtime rather than build/ABI/dependency issues?
* How many are explicit support policies rather than technical failures?
* How many create an actual pre-install candidate-selection problem?
* How many would be solved by existing mechanisms?
* How many would materially benefit from a machine-readable declaration?

The current research should therefore distinguish **examples proving existence** from **measurements proving prevalence**.

---

## 32. What evidence would falsify the proposal?

The research should define failure conditions before recommending a PEP.

The proposal becomes substantially weaker if investigation shows that:

* most apparent implementation restrictions are actually ABI or build issues;
* PEP 725 adequately covers the important build cases;
* wheel tags or variants cover the important artifact cases;
* classifiers satisfy the meaningful consumer use cases;
* implementation restrictions are too rare to justify ecosystem complexity;
* producer declarations are too unreliable for machine actionability;
* consumers do not need the information before installation;
* failed-build caching provides most of the practical benefit;
* the information is too difficult to obtain early enough;
* release-level semantics cannot accurately describe mixed artifact releases;
* conditional implementation support makes the field impractically complex.

A credible research process must allow the conclusion:

> No new standard is necessary.

---

## 33. Does the problem justify a new standard?

This remains the central question.

The current evidence supports the narrower observation:

```text
Some releases have implementation-specific support boundaries
that are not represented by their generic wheel tags or
Requires-Python.
```

The strongest examples make that observation credible.

The research has **not established**:

```text
therefore a new Core Metadata field is necessary.
```

The remaining proof obligation is:

```text
real release
    ↓
explicit support boundary
    ↓
root cause understood
    ↓
existing mechanisms insufficient
    ↓
meaningful pre-install/build consumer decision
    ↓
new machine-readable declaration provides material benefit
    ↓
declaration can be obtained early enough
    ↓
producer declarations are trustworthy enough
    ↓
benefit justifies ecosystem cost
```

Only if this chain survives adversarial investigation should a new normative metadata mechanism be recommended.

---

# Current Design Hypotheses

The following are hypotheses under investigation, not conclusions.

## A. Positive support declaration

If the research ultimately establishes that a new mechanism is justified, the current leading semantic hypothesis is a positive support declaration such as:

```toml
supported-implementation = ["cpython", "pypy"]
```

rather than:

```toml
requires-implementation = ["cpython"]
```

The reason is semantic:

```text
Requires-Implementation
    sounds like
hard installation requirement

Supported-Implementation
    sounds like
producer-declared support boundary
```

However:

> `Supported-Implementation` is a research hypothesis, not the current recommendation.

The research must first establish that a normative release-level support declaration is needed at all.

---

## B. Omission should not imply universal support

If a field were standardized, omission should not silently convert an existing distribution into either:

```text
all implementations supported
```

or:

```text
all implementations unsupported
```

The leading hypothesis is:

```text
field absent
    →
no normative implementation-support declaration
```

This preserves backwards compatibility and distinguishes unknown information from an explicit negative declaration.

This remains a hypothesis pending final semantic design.

---

## C. Support should not become a second wheel-tag system

A future implementation-support mechanism should not attempt to reproduce:

* Python version constraints;
* ABI tags;
* platform tags;
* architecture tags;
* free-threading distinctions;
* debug-build distinctions;
* arbitrary artifact variants.

The unresolved design question is whether a narrow implementation identity dimension can be defined cleanly enough to coexist with those mechanisms.

---

# Highest-Priority Remaining Questions

The research should now prioritize the questions that most directly determine whether the project should become a PEP.

1. **Root cause:** Why are the strongest real-world releases implementation-specific?

2. **Residuality:** Which cases survive PEP 725, ABI, artifact, dependency, classifier, and alternative-interpreter explanations?

3. **Consumer:** Who needs the information before installation or source build?

4. **Decision:** What concrete decision changes when the information is known?

5. **Benefit:** What measurable cost or failure does that decision avoid?

6. **Semantics:** Is the required concept support, requirement, compatibility, or something else?

7. **Trust:** Can producer declarations be maintained accurately enough for automation?

8. **Prevalence:** Is the problem common enough to justify standardization?

9. **Acquisition:** Can the information reach the intended consumer before the expensive or failing operation?

10. **Scope:** Is the declaration genuinely release-level, or does artifact-level metadata already provide the necessary semantics?

11. **Alternatives:** Could classifiers, PEP 725, wheel tags, PEP 780, PEP 825, caching, or improved interpreter compatibility solve the problem more appropriately?

12. **Placement:** If justified, does the information belong in Core Metadata, repository metadata, or another layer?

13. **Adoption:** What concrete incentive would cause projects to maintain a new field accurately?

14. **Failure mode:** What happens when a declaration is stale, incomplete, overly broad, or wrong?

15. **No-new-standard outcome:** Can the research honestly conclude that existing mechanisms are sufficient?

Until these questions are answered, the project should remain in the research phase rather than prematurely becoming a PEP proposal.
