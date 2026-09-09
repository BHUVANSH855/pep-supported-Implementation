# Decision Matrix

The research compares possible ways to represent, communicate, or act on Python implementation support.

The matrix is intentionally conservative:

> A mechanism receives a high score only for the compatibility layer it actually describes.

The matrix is a **research instrument**, not a recommendation.

The current research has established a semantic distinction between:

```text
Python version compatibility
artifact compatibility
dependency conditionality
build/host requirements
ABI/environment compatibility
producer implementation support
observed installation/build failure
```

The central decision is therefore not merely:

```text
Can "CPython only" be represented?
```

It is:

```text
Does producer-declared implementation support constitute
a sufficiently objective, stable, prevalent, and
consumer-useful fact that packaging standards should make
it normative?
```

---

# 1. Evaluation criteria

| Criterion               | Meaning                                                                                                       |
| ----------------------- | ------------------------------------------------------------------------------------------------------------- |
| Semantic fit            | Does the mechanism describe the actual fact being investigated?                                               |
| Release-level           | Can it describe a project release rather than only one artifact?                                              |
| Sdist applicability     | Can it help before building source?                                                                           |
| Resolver utility        | Can an installer use it for candidate selection?                                                              |
| Existing adoption       | Can current metadata/tooling be reused without requiring new publisher behavior?                              |
| Backwards compatibility | Can existing packages continue unchanged?                                                                     |
| False-positive risk     | Could tooling incorrectly classify a technically incompatible/compatible release?                             |
| Stale-declaration risk  | Could incorrect producer metadata cause valid candidates to be rejected or invalid candidates to be accepted? |
| Publisher burden        | How difficult is the mechanism to create and maintain correctly?                                              |
| Ecosystem breadth       | Is it useful across pip, uv, downstreams, indexes, build frontends, and other tooling?                        |
| Layer separation        | Does it avoid conflating ABI, artifact, build, dependency, and support semantics?                             |
| Evidence strength       | How directly does current empirical evidence support the mechanism as a solution?                             |
| Semantic precision      | Can its normative meaning be stated precisely enough for interoperable tooling?                               |
| Transport availability  | Can consumers obtain the information early enough to use it for candidate selection?                          |
| First-time benefit      | Can it avoid a failure before the first build/install attempt rather than only after observation?             |

The last three criteria are particularly important.

A mechanism can be semantically appropriate but still fail to provide enough information early enough for a resolver to benefit.

---

# 2. Candidate 1 — Existing Trove classifiers

Example:

```text
Programming Language :: Python :: Implementation :: CPython
```

## Strengths

* existing standardized vocabulary;
* already published by projects;
* human-readable;
* useful for PyPI display and search;
* communicates positive support information;
* requires no new metadata field;
* can communicate a producer's declared implementation position.

## Weaknesses

Current classifier semantics are descriptive rather than a standardized hard compatibility predicate.

In particular:

```text
CPython classifier present
        ≠
all other implementations unsupported
```

and:

```text
PyPy classifier absent
        ≠
PyPy incompatible
```

The existing classifier model therefore has an important open-world property:

```text
listed implementation
    → explicitly classified

unlisted implementation
    → support unspecified
```

rather than:

```text
listed implementation
    → compatible

unlisted implementation
    → incompatible
```

Changing this interpretation would create a major historical compatibility problem.

## Empirical finding

A controlled pip experiment constructed:

```text
Name: classifier-only-cpython-test
Version: 1.0.0
Requires-Python: >=3.8
Classifier:
    Programming Language :: Python :: Implementation :: CPython
Wheel:
    py3-none-any
```

pip was asked to resolve the candidate for a PyPy target and selected the generic wheel.

This demonstrates:

```text
CPython Trove classifier
        ≠
normative pip implementation exclusion
```

The experiment does **not** establish that a new field is necessary.

It establishes only that current implementation classifiers cannot simply be assumed to provide hard implementation filtering.

## Verdict

**Most important existing alternative.**

Classifiers should remain the baseline against which every proposed normative mechanism is compared.

The research must prove why positive/descriptive implementation information is insufficient for the actual consumer decision.

---

# 3. Candidate 2 — Reinterpret classifiers as hard constraints

## Strengths

* no new field;
* implementation vocabulary already exists;
* existing metadata is theoretically available to tooling;
* potentially resolver-visible without introducing another syntax.

## Problems

Existing classifiers were not specified as exhaustive compatibility constraints.

A historical release such as:

```text
Classifier:
    Programming Language :: Python :: Implementation :: CPython
```

does not necessarily establish:

```text
PyPy = incompatible
```

Changing classifier semantics could therefore produce:

```text
old release
+
missing PyPy classifier
        ↓
new resolver
        ↓
PyPy rejected
```

even when the package works on PyPy.

The change would also make metadata omissions materially dangerous.

## Verdict

**High compatibility and semantic risk.**

A positive/advisory interpretation is substantially safer than retroactively turning classifier omissions into hard incompatibility.

---

# 4. Candidate 3 — `Requires-Implementation`

Example:

```text
Requires-Implementation: cpython
```

## Strengths

* parallels `Requires-Python`;
* naturally suggests machine-actionable requirements;
* easy for resolvers to conceptualize;
* could express a hard candidate constraint.

## Problems

The name is stronger than the current evidence.

It naturally suggests:

```text
the environment MUST provide this implementation
```

rather than:

```text
the producer declares this implementation supported.
```

This creates several unresolved questions:

* Does it describe runtime implementation?
* Does it describe the build implementation?
* Does it describe the host implementation?
* Does it mean technically required?
* Does it mean officially supported?
* Does it mean guaranteed?
* Can a compatible implementation satisfy it?
* How are implementation versions expressed?
* How are multiple implementations represented?

The distinction is fundamental:

```text
requires CPython
```

versus:

```text
supports CPython
```

The first implies necessity.

The second describes producer support.

Current evidence contains both technical restrictions and explicit support-policy boundaries, so they should not automatically receive identical normative treatment.

## Verdict

**Currently not the preferred design.**

The name and semantics are too requirement-oriented for the evidence currently available.

---

# 5. Candidate 4 — `Supported-Implementation`

Example:

```text
Supported-Implementation: cpython
Supported-Implementation: pypy
```

## Strengths

* describes producer support rather than technical necessity;
* naturally supports multiple implementations;
* preserves the distinction between support and requirement;
* potentially useful for sdist candidate selection;
* corresponds more closely to support-policy cases such as RestrictedPython;
* can coexist with `Requires-Python`.

## Problems

The central word — `supported` — requires a precise normative definition.

Possible meanings include:

```text
technically compatible
officially supported
tested
guaranteed
safe
maintained
```

These are not equivalent.

Additional unresolved questions include:

* What does omission mean?
* Is the list exhaustive?
* Is absence equivalent to incompatibility?
* Can support be conditional?
* Does the field describe runtime or build implementation?
* Must the value be identical across sdist and wheels?
* How does it interact with wheel tags?
* How does it interact with implementation versions?
* What happens when a declaration becomes stale?

The field also risks mixing:

```text
technical compatibility
```

with:

```text
producer support policy
```

unless the specification explicitly chooses one semantic model.

## Residual evidence

The current cases demonstrate different kinds of boundaries:

```text
HAX
    explicit runtime rejection

RestrictedPython
    explicit support/security boundary

simple-ctx-log
    private CPython API dependence

TribeCore
    possible native/build/ABI/component causes
```

These should not automatically receive identical normative treatment.

## Verdict

**Strong semantic hypothesis, but not yet justified as a standard.**

It should remain under investigation rather than being presented as the research conclusion.

---

# 6. Candidate 5 — Wheel compatibility tags

Wheel compatibility tags are the established artifact-level mechanism.

Examples include:

```text
cp312
pp312
py3
```

combined with ABI and platform tags.

## Strengths

* mature;
* precise for built artifacts;
* already resolver-relevant;
* implementation-aware;
* can express implementation-specific artifact compatibility;
* does not require a new release-level support declaration when the restriction is intrinsic to the artifact.

PEP 425 explicitly defines the Python tag as indicating the implementation and version required by a built distribution. It also deliberately keeps compatibility tags outside `METADATA`, because `METADATA` should describe the distribution rather than one particular build.

## Weakness

Wheel tags describe the **artifact**, not necessarily the producer's release-wide support policy.

For example:

```text
release 1.0
    ├── py3-none-any wheel
    └── sdist

producer:
    CPython-only
```

The generic wheel tag does not communicate the producer's separate release-level support policy.

Conversely:

```text
cp312-...
```

does not automatically prove that every artifact or the entire source release is CPython-only.

## Verdict

**Established artifact-level mechanism.**

Use it whenever the restriction is genuinely an artifact compatibility property.

Do not create a release-level support field merely to duplicate wheel compatibility information.

---

# 7. Candidate 6 — PEP 825 wheel variants

## Strengths

* richer artifact compatibility;
* resolver-oriented;
* index-level variant information;
* designed for selecting among multiple compatible artifacts;
* can represent additional environment/feature properties.

## Weakness

PEP 825 is an artifact-selection mechanism.

Its variant properties describe compatibility requirements of particular wheel variants.

It does not by itself answer:

```text
Does the producer support PyPy for this release?
```

A release-level producer-support statement and an artifact compatibility predicate remain different concepts.

## Verdict

**Complementary, not a direct replacement.**

If a proposed residual case can be solved by expressing an artifact property through wheel tags or variants, it should not be counted as evidence for release-level Core Metadata.

---

# 8. Candidate 7 — PEP 508 environment markers

## Strengths

* implementation identity already exists;
* standardized vocabulary;
* conditional dependencies are established;
* resolver-aware;
* implementation-specific dependencies can be expressed.

Examples include markers based on:

```text
implementation_name
implementation_version
platform_python_implementation
```

## Weakness

A dependency marker answers:

```text
When is dependency X required?
```

It does not directly answer:

```text
Is distribution X itself supported?
```

For example:

```text
Requires-Dist:
    dependency-a;
    implementation_name == "cpython"
```

means the dependency applies conditionally.

It does not necessarily mean:

```text
the containing distribution is unsupported on PyPy.
```

The current dependency specification explicitly defines a false environment marker as causing the dependency to be ignored.

## Verdict

**Necessary environment machinery, not a release-support declaration.**

A case solvable through conditional dependencies should not be counted as a residual implementation-support case.

---

# 9. Candidate 8 — PEP 780 ABI/environment features

## Strengths

* handles ABI/environment dimensions;
* avoids turning implementation identity into an overloaded compatibility language;
* provides a more precise mechanism for environment-level distinctions;
* useful for cases such as free-threaded versus traditional CPython.

## Weakness

ABI/environment compatibility is not the same as producer support policy.

A release can be:

```text
ABI-compatible
```

while:

```text
producer-supported only on CPython
```

Conversely, an apparent implementation restriction may actually be caused by:

```text
ABI
platform
interpreter build configuration
```

and therefore belong in an existing artifact/environment mechanism.

## Verdict

**Complementary and an important scope boundary.**

ABI/environment cases must be removed from the implementation-support residual set before a new field is justified.

---

# 10. Candidate 9 — PEP 725 external dependency metadata

## Strengths

* addresses external dependency information;
* distinguishes build/host concerns;
* directly relevant to source builds;
* can prevent external dependency failures from being misclassified as implementation incompatibility.

## Weakness

A build requirement is not automatically a runtime support declaration.

For example:

```text
requires external library X to build
```

does not imply:

```text
supports CPython only
```

A package may support:

```text
CPython
PyPy
GraalPy
```

while requiring an unavailable compiler or external library to build.

## Verdict

**First-class alternative for build cases.**

Any candidate whose apparent implementation restriction is actually caused by a missing build/host/external dependency must be excluded from the residual implementation-support evidence.

---

# 11. Candidate 10 — Source-build policy

## Strengths

* directly targets unwanted or expensive source builds;
* may prevent immediate operational failures;
* can be useful where binary artifacts are unavailable.

## Weakness

It answers:

```text
Should an installer build this source?
```

rather than:

```text
Which Python implementations does the producer support?
```

These questions overlap operationally but are not semantically identical.

A project may legitimately be:

```text
supported on PyPy
```

while requiring:

```text
manual/source build
```

Likewise, a source-build policy could prevent a build even when the build would technically succeed.

## Direct prior art

The 2024 Packaging discussion on preventing unwanted sdist builds considered a mechanism specifically aimed at avoiding automatic source builds.

This is strong evidence that:

```text
automatic sdist-build avoidance
```

is a recognized problem in its own right.

It should therefore be treated as an alternative design layer rather than as proof that implementation-support metadata is necessary.

## Verdict

**Adjacent problem, not equivalent semantics.**

---

# 12. Candidate 11 — Failed-build caching

## Strengths

* requires no new package metadata;
* can prevent repeated expensive failures;
* directly addresses repeated operational failures;
* imposes no publisher adoption requirement;
* learns from actual environment-specific observations.

## Weakness

A cached failure:

* is discovered only after a build attempt;
* is environment-specific;
* may become stale;
* does not communicate producer support intent;
* may not transfer safely between environments;
* does not help the first resolver avoid the initial build.

The semantic distinction is:

```text
metadata:
    producer declaration about a release

failure cache:
    consumer observation about an attempted build
```

These are not interchangeable.

## Critical comparison

The research should test whether failed-build caching provides enough practical benefit that a standardized declaration is unnecessary.

A metadata proposal needs to demonstrate a concrete advantage such as:

```text
first-time avoidance
        +
cross-environment knowledge
        +
producer-declared support information
        +
candidate selection before build
```

rather than merely showing that caching is imperfect.

## Verdict

**Serious competing operational solution.**

It should remain in the final comparison.

---

# 13. Candidate 12 — No new standard

## Strengths

* zero new metadata burden;
* avoids premature standardization;
* allows tooling to combine existing mechanisms;
* preserves descriptive classifier semantics;
* avoids stale producer declarations becoming resolver constraints;
* keeps compatibility layers separate.

## Weakness

The strongest residual candidates remain awkward if no existing mechanism can make the relevant support fact machine-actionable before source build or runtime.

## Verdict

**Fully credible final outcome.**

The research must remain willing to conclude:

```text
no new standard
```

or:

```text
improve existing tooling/mechanisms
```

rather than treating a new field as the expected result.

---

# 14. Important distinction — technical compatibility versus support policy

The residual cases currently reveal at least two distinct concepts.

## Technical compatibility

```text
this release cannot operate correctly on PyPy
```

This is a statement about technical behavior.

## Producer support policy

```text
this project officially supports CPython only
```

This is a statement about producer policy.

These can coincide:

```text
technical incompatibility
        +
support policy
```

but they do not have to.

A package may:

```text
technically work on PyPy
```

while:

```text
the producer does not officially support PyPy
```

Conversely, a producer may claim PyPy support while an implementation-specific bug makes the release fail.

This distinction is critical because a normative resolver constraint is much more consequential if it means:

```text
known incompatibility
```

than if it means:

```text
producer chooses not to support an otherwise functioning environment
```

A future standard must therefore decide exactly what kind of fact it represents.

---

# 15. Important distinction — positive support versus negative exclusion

The existing classifier system naturally communicates:

```text
CPython is explicitly supported/classified
```

A normative compatibility mechanism may instead communicate:

```text
non-CPython candidates are incompatible
```

These are not equivalent.

Formally:

```text
positive support declaration
        ≠
closed compatibility set
```

This is one of the strongest arguments against simply reinterpreting classifiers.

It is also one of the most important semantic questions for `Supported-Implementation`.

---

# 16. Important distinction — release versus artifact

The research should maintain the following boundary:

```text
release-level fact
        ≠
artifact-level fact
```

Examples:

```text
wheel:
    cp312-cp312-manylinux_2_17_x86_64

    → artifact compatibility


release:
    producer supports CPython only

    → potentially release-level support
```

PEP 425 deliberately keeps wheel compatibility tags separate from `METADATA`, because compatibility tags describe a particular built artifact.

Therefore a proposed Core Metadata field should not duplicate artifact compatibility.

---

# 17. Important distinction — source build versus implementation support

A source build can fail because of:

```text
implementation
Python version
platform
ABI
compiler
linker
external library
SDK
build backend
dependency
configuration
temporary defect
```

Only some of these are implementation-support problems.

Therefore:

```text
sdist build failed on PyPy
```

is not sufficient evidence.

The research must identify the root cause before entering a case into the residual set.

---

# 18. Important distinction — metadata transport versus metadata semantics

Simple API metadata transport provides a way to obtain Core Metadata before downloading the complete distribution when the repository exposes it.

Current Simple API semantics make Core Metadata availability optional.

Therefore:

```text
metadata can be transported early
```

does not imply:

```text
every metadata field is a resolver constraint
```

nor:

```text
metadata is always available early.
```

A future implementation-support field would need both:

```text
semantic justification
+
consumer-facing transport/availability behavior
```

---

# 19. Stale-declaration risk

Any release-level support field introduces stale metadata risk.

## False negative

```text
package becomes compatible
        ↓
producer forgets to update metadata
        ↓
resolver rejects valid candidate
```

## False positive

```text
package becomes incompatible
        ↓
producer forgets to update metadata
        ↓
resolver accepts candidate
        ↓
failure occurs later
```

This risk is particularly important for support declarations because technical compatibility can change without the project's metadata being automatically regenerated.

A future field would therefore need:

* clear release semantics;
* clear publisher responsibility;
* sdist/wheel consistency rules;
* guidance for support transitions;
* explicit behavior for missing metadata;
* and careful consideration of whether a producer declaration should be authoritative enough to cause rejection.

---

# 20. Publisher burden

A new field introduces an additional release-maintenance requirement.

A publisher would need to determine:

```text
Which implementations are supported?
```

and potentially maintain:

```text
support set
+
version boundaries
+
release changes
```

This is more burdensome than simply publishing a classifier if the field has normative consequences.

The burden is justified only if the information provides a meaningful consumer benefit.

A proposal should therefore avoid requiring publishers to make distinctions that are inherently uncertain, such as:

```text
tested
but not supported
```

unless those semantics are explicitly defined.

---

# 21. Ecosystem adoption

A normative field is only useful to the extent that:

```text
publishers declare it
        +
indexes transport it
        +
resolvers consume it
```

If adoption is low:

```text
new field absent
        ↓
implementation support unknown
```

and the resolver may still need to attempt the same source build.

This creates an important question:

> Is the expected consumer benefit large enough to justify publisher adoption, given that absence of the field cannot safely be interpreted as universal compatibility?

Classifier adoption and consistency should therefore be measured before assuming that a new field would be widely populated.

---

# 22. Current qualitative matrix

The scores are intentionally provisional.

They are not mathematical measurements.

| Solution                   | Semantic fit                 | Release-level     | Sdist              | Resolver              | Existing adoption | Backwards compatibility | False-positive risk | Stale risk | Publisher burden | Ecosystem breadth | Layer separation | Evidence strength            | Semantic precision | Transport / early use          | Current posture                    |
| -------------------------- | ---------------------------- | ----------------- | ------------------ | --------------------- | ----------------- | ----------------------- | ------------------- | ---------- | ---------------- | ----------------- | ---------------- | ---------------------------- | ------------------ | ------------------------------ | ---------------------------------- |
| Trove classifiers          | High                         | High              | Medium             | Low/unknown           | High              | High                    | Low/Medium          | Medium     | Low              | High              | High             | **High for descriptive use** | Medium             | Medium                         | **Investigate first**              |
| Reinterpreted classifiers  | Medium                       | High              | Medium             | High                  | High              | Low                     | High                | High       | Low initially    | High              | Medium           | Low                          | Low/Medium         | Medium                         | **Risky**                          |
| `Requires-Implementation`  | High                         | High              | High               | High                  | Low               | High                    | Medium/High         | High       | Medium           | High              | Medium           | Low/Medium                   | Medium             | High                           | **Not preferred**                  |
| `Supported-Implementation` | High                         | High              | High               | High                  | Low               | High                    | Medium              | High       | Medium           | High              | Medium/High      | **Medium**                   | Low/Medium         | High                           | **Hypothesis**                     |
| Wheel tags                 | High for artifact            | Low for release   | Low                | High                  | High              | High                    | Low                 | Low        | Low              | High              | High             | **High**                     | High               | High                           | **Established artifact mechanism** |
| PEP 825 variants           | High for artifact            | Low               | Low                | High                  | Emerging          | High                    | Medium              | Low        | Medium           | Emerging          | High             | Medium                       | Medium/High        | High                           | **Complementary**                  |
| PEP 508                    | High for dependencies        | Medium            | Medium             | High for dependencies | High              | High                    | Low                 | Low        | Low              | High              | High             | **High**                     | High               | High                           | **Complementary**                  |
| PEP 780                    | High for ABI/environment     | Medium            | Medium             | High for ABI/deps     | Emerging          | High                    | Low                 | Low        | Medium           | Emerging          | High             | **High for ABI cases**       | High               | High                           | **Scope boundary**                 |
| PEP 725                    | High for build/external deps | High              | High               | Emerging              | Emerging          | High                    | Medium              | Medium     | Medium           | Emerging          | High             | **High for build cases**     | High               | Medium/High                    | **Test first for build cases**     |
| Source-build policy        | Medium                       | High              | High               | High operationally    | Low               | High                    | Medium              | Medium     | Medium           | Medium            | Medium           | Medium                       | Medium             | High                           | **Adjacent**                       |
| Failed-build cache         | Low for semantics            | Environment-level | High operationally | Medium                | High              | High                    | Low                 | Medium     | Low              | Medium            | High             | Medium                       | High operationally | Low first-time                 | **Serious alternative**            |
| No new standard            | Medium                       | Medium            | Medium             | Medium                | High              | Highest                 | Lowest              | Lowest     | Lowest           | Highest           | Highest          | **Always viable**            | High               | Depends on existing mechanisms | **Credible outcome**               |

---

# 23. Interpretation

Several conclusions are now reasonably well supported.

## 23.1 There is a genuine semantic distinction

The research has identified cases where:

```text
producer support boundary
```

and:

```text
artifact compatibility
```

do not necessarily coincide.

HAX is a strong technical example:

```text
explicit CPython-only runtime enforcement
+
py3-none-any wheel
```

RestrictedPython provides a different kind of example:

```text
explicit CPython-only security/support boundary
```

These demonstrate that:

```text
generic artifact
        ≠
universal producer support
```

However, this establishes a **semantic distinction**, not yet the necessity of a new standard.

---

# 24. Existing classifiers are not equivalent to normative compatibility

The controlled pip experiment establishes that a CPython implementation classifier does not itself prevent a generic wheel from being selected for a PyPy target.

Therefore:

```text
classifier
    ≠
existing normative resolver constraint
```

This is useful evidence.

It does **not** establish:

```text
new field
    = correct solution
```

The missing step is consumer benefit.

---

# 25. Artifact mechanisms remain correct for artifact-derived restrictions

If a restriction is caused by:

* ABI;
* compiled extension compatibility;
* platform;
* interpreter-specific binary interface;
* CPU feature;
* another artifact property;

then artifact compatibility mechanisms remain preferable.

A release-level support field should not become:

```text
a second ABI/tag language
```

or duplicate information already encoded by wheel compatibility.

---

# 26. PEP 508 remains the dependency boundary

If the actual problem is:

```text
dependency X required on CPython
dependency Y required on PyPy
```

then environment markers are the existing mechanism.

The residual set should contain only cases where:

```text
the distribution itself
```

has an implementation-support restriction that dependency metadata cannot represent.

---

# 27. PEP 725 must eliminate build-only cases

If a package fails because:

```text
compiler unavailable
external library unavailable
SDK unavailable
host dependency unavailable
```

the case should not be counted as implementation-support evidence.

The research should classify these failures as build/host/environment cases first.

---

# 28. Support policy is the hardest semantic question

The current evidence shows that:

```text
supported
```

can mean more than:

```text
technically executable
```

RestrictedPython is especially important here because the producer's CPython-only position is tied to a security boundary.

A future field must therefore answer whether:

```text
Supported-Implementation: cpython
```

means:

1. technically compatible;
2. officially supported;
3. guaranteed to work;
4. tested;
5. safe;
6. maintained;
7. or some explicitly defined combination.

Without that definition, resolver semantics would be unsafe.

---

# 29. The critical missing evidence

The matrix currently shows a plausible semantic hypothesis but not a sufficient empirical case.

The research still needs to establish:

```text
How many real releases have the residual property?
```

and:

```text
How often does the property affect a real resolver decision?
```

and:

```text
How often would knowing it before build avoid meaningful work?
```

and:

```text
Would publishers actually provide reliable declarations?
```

and:

```text
Would a new field outperform failed-build caching or other tooling?
```

The proposal should not infer prevalence from a handful of interesting packages.

---

# 30. Decision gate

A new release-level implementation field should not be recommended merely because:

```text
it is semantically elegant
```

or because:

```text
several packages exhibit CPython-only behavior.
```

The research should recommend a new standard only if the following conjunction is substantially demonstrated:

```text
real residual cases
        +
root cause belongs in release-level support metadata
        +
existing artifact mechanisms are insufficient
        +
Requires-Python is insufficient
        +
PEP 508 is insufficient for package self-support
        +
PEP 725/build metadata are insufficient for build cases
        +
classifiers are insufficient for the required consumer decision
        +
source-build policy is insufficient
        +
failed-build caching is materially insufficient
        +
a concrete pre-install/pre-build decision changes
        +
consumer benefit is measurable
        +
cases are sufficiently prevalent/generalizable
        +
publisher burden is credible
        +
stale-declaration risk is acceptable
        +
semantics can be stated precisely
        +
technical compatibility and support policy can be distinguished
```

If the central semantic or empirical conditions fail, the research should instead conclude:

```text
no new standard
```

or, where appropriate:

```text
improve existing tooling/mechanisms
```

---

# 31. Current decision posture

The matrix currently supports this position:

```text
Semantic distinction:
    established

Existing implementation classifiers:
    descriptive and useful,
    but not normative resolver constraints

Reinterpreting classifiers:
    high compatibility/semantic risk

Requires-Implementation:
    plausible but requirement-oriented;
    currently not preferred

Supported-Implementation:
    strongest semantic hypothesis;
    not yet justified

Requires-Python:
    version compatibility only

Wheel tags:
    established artifact compatibility mechanism

PEP 825:
    complementary artifact-selection mechanism

PEP 508:
    dependency conditionality;
    not package-self-support metadata

PEP 780:
    ABI/environment boundary

PEP 725:
    important build/external-dependency boundary

Simple API Core Metadata:
    enables early metadata access when available;
    does not establish semantics

Source-build policy:
    adjacent operational problem

Failed-build caching:
    serious operational alternative

No new standard:
    fully credible final outcome
```

---

# 32. Current research conclusion

The research should **not yet select a metadata design**.

The evidence has established that implementation identity and producer implementation support are not perfectly represented by today's packaging mechanisms.

It has **not yet established** that the missing representation should be a new Core Metadata field.

The strongest unresolved question is now:

```text
Are there sufficiently prevalent, reproducible, release-level
implementation-support cases where:

    1. the producer's implementation boundary is real;
    2. the restriction is not merely artifact/build/dependency/ABI;
    3. existing metadata cannot express the relevant fact;
    4. a resolver can make a materially better decision from it;
    5. the decision must happen before build/install;
    6. the declaration can be made reliably at release time;
    7. and the benefit outweighs publisher and ecosystem costs?
```

If yes, the next design question is likely:

```text
Supported-Implementation
```

versus:

```text
Requires-Implementation
```

with `Supported-Implementation` currently having the stronger semantic fit.

If no, the appropriate conclusion may be:

```text
retain classifiers
+
improve tooling
+
improve failure handling
+
use existing artifact/build/dependency mechanisms
```

rather than introducing new Core Metadata.

That is the decision the remaining empirical research must answer.
