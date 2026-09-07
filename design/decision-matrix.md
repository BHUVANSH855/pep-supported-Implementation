# Decision Matrix

The research compares possible ways to represent or act on Python
implementation support.

The matrix is intentionally conservative: a mechanism receives a high score
only for the layer it actually describes.

## Evaluation criteria

| Criterion               | Meaning                                                                   |
| ----------------------- | ------------------------------------------------------------------------- |
| Semantic fit            | Does the mechanism describe the actual fact?                              |
| Release-level           | Can it describe a project version rather than one artifact?               |
| Sdist applicability     | Can it help before building source?                                       |
| Resolver utility        | Can an installer use it for candidate selection?                          |
| Existing adoption       | Can current metadata be reused?                                           |
| Backwards compatibility | Can existing packages continue unchanged?                                 |
| False-positive risk     | Can tooling safely distinguish support from implementation-specific code? |
| Publisher burden        | How difficult is it to maintain?                                          |
| Ecosystem breadth       | Useful to pip/uv/downstreams/indexes/etc.?                                |
| Layer separation        | Does it avoid conflating ABI, artifact, build, and support semantics?     |

---

## Candidate 1 — existing Trove classifiers

Example:

```text
Programming Language :: Python :: Implementation :: CPython
```

### Strengths

* already published by projects;
* established vocabulary;
* human-readable;
* no new metadata field;
* can communicate positive support information.

### Weaknesses

* current semantics are descriptive;
* no standardized candidate rejection rule;
* absence is ambiguous;
* existing classifier usage may not have been intended as an exhaustive
  compatibility set;
* changing semantics could affect old metadata.

### Research question

Can the classifier safely be interpreted as:

> The publisher explicitly supports this implementation.

without interpreting absence as unsupported?

### Verdict

**Most important existing alternative. Unresolved.**

---

## Candidate 2 — reinterpret classifiers as hard constraints

### Strengths

* no new field;
* immediately available to tools.

### Problems

Existing classifiers were not specified as exhaustive compatibility
constraints.

Changing their semantics could turn descriptive metadata into installation
rejection rules.

### Verdict

**High compatibility risk.**

A positive/advisory interpretation is substantially more plausible than
retroactively turning all classifier omissions into hard incompatibility.

---

## Candidate 3 — `Requires-Implementation`

Example:

```text
Requires-Implementation: cpython
```

### Strengths

* parallels `Requires-Python`;
* direct resolver semantics;
* easy to explain.

### Problems

* sounds like a technical requirement rather than support policy;
* risks conflating build-time and runtime implementation requirements;
* can create stale negative claims;
* unclear treatment of implementation forks;
* stronger semantics than the current evidence justifies.

### Verdict

**Currently not the preferred design.**

---

## Candidate 4 — `Supported-Implementation`

Example:

```text
Supported-Implementation: cpython
Supported-Implementation: pypy
```

### Strengths

* describes producer support rather than technical necessity;
* naturally fits multiple implementations;
* positive support semantics are less aggressive than negative exclusion;
* could help describe releases before an sdist build.

### Problems

* still requires a precise definition of "supported";
* omission semantics must be explicit;
* publisher adoption may be poor;
* incorrect declarations can affect resolution;
* does not solve build/ABI/private-API questions automatically.

### Verdict

**Strong semantic hypothesis, but not yet justified as a standard.**

---

## Candidate 5 — wheel tags / PEP 425

### Strengths

* mature;
* precise for built artifacts;
* already resolver-relevant.

### Weakness

It describes the artifact, not necessarily the producer's release-wide support
policy.

A release can contain:

```text
py3-none-any
+
CPython-only producer statement
```

### Verdict

**Keep as the artifact-level mechanism; not a complete replacement.**

---

## Candidate 6 — PEP 825 wheel variants

### Strengths

* richer artifact compatibility;
* index-level variant information;
* resolver-oriented.

### Weakness

PEP 825 is about wheel variants, not a general declaration that a release
supports an implementation.

### Verdict

**Complementary, not a direct replacement.**

---

## Candidate 7 — PEP 508 markers

### Strengths

* implementation identity is already available;
* conditional dependencies are well established.

### Weakness

A dependency marker answers:

```text
When is dependency X required?
```

It does not directly answer:

```text
Is distribution X itself supported?
```

### Verdict

**Necessary environment machinery, not a release-support declaration.**

---

## Candidate 8 — PEP 780 ABI features

### Strengths

* handles free-threading and other ABI dimensions;
* prevents implementation name from becoming an overloaded compatibility
  language.

### Weakness

It describes ABI/environment dimensions rather than producer support policy.

### Verdict

**Complementary and an important scope boundary.**

---

## Candidate 9 — PEP 725 external dependency metadata

### Strengths

* addresses build/host/runtime external dependency information;
* explicitly distinguishes build machine and host machine;
* directly relevant to source-build restrictions.

### Weakness

A build requirement is not automatically a runtime support declaration.

The research must therefore classify a candidate as build-only before using it as
evidence for release-level implementation support.

### Verdict

**First-class alternative for build cases; not currently equivalent to
release support metadata.**

---

## Candidate 10 — source-build policy

### Strengths

* directly targets expensive/failing source builds;
* may avoid an immediate operational failure.

### Weakness

It answers:

```text
Should an installer build this source?
```

rather than:

```text
Which Python implementations does the producer support?
```

### Verdict

**Adjacent problem, not equivalent semantics.**

---

## Candidate 11 — failed-build caching

### Strengths

* requires no new package metadata;
* can prevent repeated expensive failures;
* directly addresses the operational problem of repeated source builds;
* imposes no publisher adoption requirement.

### Weakness

A cached failure:

* is discovered only after a build attempt;
* is environment-specific;
* may become stale when implementations or packages are fixed;
* does not communicate producer support intent;
* may not transfer between machines or environments;
* does not help a first-time resolver avoid the initial build.

### Critical comparison

The research must test whether:

```text
failed-build cache
```

provides enough practical benefit that a standardized declaration is unnecessary.

The metadata proposal must demonstrate a concrete advantage such as:

```text
first-time avoidance
+
cross-environment knowledge
+
producer-declared support intent
+
candidate selection before build
```

rather than merely arguing that caching is imperfect.

### Verdict

**Serious competing operational solution. Must be experimentally compared.**

---

## Candidate 12 — no new standard

### Strengths

* zero new metadata burden;
* avoids premature standardization;
* lets tooling combine existing mechanisms.

### Weakness

The strongest residual candidates remain awkward if no existing mechanism can
make the support fact machine-actionable before source build.

### Verdict

**Still a credible final outcome. The research must be willing to choose it.**

---

## Current matrix

| Solution                 |                 Semantic fit |              Sdist |          Resolver | Existing | False-positive risk | Current posture                |
| ------------------------ | ---------------------------: | -----------------: | ----------------: | -------: | ------------------: | ------------------------------ |
| Trove classifier         |                         High |             Medium |       Low/unknown |     High |          Low/Medium | **investigate first**          |
| Reinterpreted classifier |                       Medium |             Medium |              High |     High |                High | risky                          |
| Requires-Implementation  |                         High |               High |              High |      Low |              Medium | not preferred                  |
| Supported-Implementation |                         High |               High |              High |      Low |              Medium | **hypothesis**                 |
| Wheel tags               |            High for artifact |                Low |              High |     High |                 Low | established                    |
| PEP 825 variants         |            High for artifact |                Low |              High | Emerging |              Medium | complementary                  |
| PEP 508                  |                High for deps |             Medium |     High for deps |     High |                 Low | complementary                  |
| PEP 780                  |                 High for ABI |             Medium | High for ABI/deps | Emerging |                 Low | complementary                  |
| PEP 725                  | High for build/external deps |               High |          Emerging | Emerging |              Medium | **test first for build cases** |
| Source-build policy      |                       Medium |               High |              High |      Low |              Medium | adjacent                       |
| Failed-build cache       |            Low for semantics | High operationally |            Medium |     High |                 Low | **serious alternative**        |
| No new standard          |                       Medium |             Medium |            Medium |     High |              Lowest | **credible outcome**           |

---

## Decision gate

A new release-level field should not be recommended merely because it is
semantically elegant.

The research should recommend a new standard only if:

```text
real residual cases
+
root cause belongs in package support metadata
+
classifiers are insufficient
+
PEP 725/build metadata are insufficient for build cases
+
failed-build caching is materially insufficient
+
pre-install candidate selection changes
+
consumer benefit is measurable
+
publisher burden is credible
+
semantics can be stated precisely
```

Otherwise the result should be:

```text
no new standard
```

This matrix is a research instrument, not a recommendation.
