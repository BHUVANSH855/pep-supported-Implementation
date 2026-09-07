# Decision Matrix

The research is comparing several possible ways to represent or act on
Python implementation support.

## Evaluation criteria

| Criterion | Meaning |
|---|---|
| Semantic fit | Does the mechanism describe the actual fact? |
| Release-level | Can it describe a project version rather than one artifact? |
| Sdist applicability | Can it help before building source? |
| Resolver utility | Can an installer use it for candidate selection? |
| Existing adoption | Can current metadata be reused? |
| Backwards compatibility | Can existing packages continue unchanged? |
| False-positive risk | Can tooling safely distinguish support from implementation-specific code? |
| Publisher burden | How difficult is it to maintain? |
| Ecosystem breadth | Useful to pip/uv/downstreams/indexes/etc.? |
| Layer separation | Does it avoid conflating ABI, artifact, build, and support semantics? |

---

## Candidate 1 — existing Trove classifiers

Example:

```text
Programming Language :: Python :: Implementation :: CPython
```

### Strengths

- already published by projects;
- established vocabulary;
- human-readable;
- no new metadata field;
- explicitly discussed as a support signal in the January 2024 thread.

### Weaknesses

- descriptive rather than normative;
- no standardized candidate rejection rule;
- absence is ambiguous;
- existing classifier usage may not have been intended as an exhaustive
  compatibility set.

### Verdict

**Strong existing solution for signaling; unresolved for normative resolution.**

---

## Candidate 2 — reinterpret classifiers as hard constraints

### Strengths

- no new field;
- immediately available to tools.

### Problems

Existing classifiers were not specified as exhaustive compatibility
constraints.

Changing their semantics could turn existing descriptive metadata into
installation rejection rules.

### Verdict

**High compatibility risk.**

---

## Candidate 3 — `Requires-Implementation`

Example:

```text
Requires-Implementation: cpython
```

### Strengths

- parallels `Requires-Python`;
- direct resolver semantics;
- easy to explain.

### Problems

- sounds like a technical requirement rather than support policy;
- potentially creates stale negative claims;
- unclear treatment of implementation forks;
- likely to be confused with build-time implementation requirements.

### Verdict

**Semantically attractive but currently not preferred.**

---

## Candidate 4 — `Supported-Implementation`

Example:

```text
Supported-Implementation: cpython
Supported-Implementation: pypy
```

### Strengths

- describes producer support rather than technical necessity;
- naturally fits multiple implementations;
- avoids claiming that unlisted implementations are mathematically
  impossible;
- matches the strongest empirical interpretation of the residual cases.

### Problems

- if normative, tools still need to define what omission means;
- if a project forgets to update it, a valid installation may be rejected;
- support is a stronger concept than mere importability;
- needs a precise relationship to implementation identity.

### Verdict

**Best semantic candidate so far, but not yet justified as a standard.**

---

## Candidate 5 — wheel tags / PEP 425

### Strengths

- mature;
- precise for built artifacts;
- already resolver-relevant.

### Weakness

It describes the artifact.

The residual cases demonstrate that:

```text
py3-none-any
```

can coexist with:

```text
CPython only
```

at the producer-support level.

### Verdict

**Keep as artifact-level mechanism; not a complete replacement.**

---

## Candidate 6 — PEP 825 wheel variants

### Strengths

- richer artifact compatibility;
- index-level variant information;
- resolver-oriented.

### Weakness

PEP 825 is about wheel variants, not a general declaration that a release
supports or does not support an implementation.

It should not be used to force release policy into artifact metadata.

### Verdict

**Complementary, not a direct replacement.**

---

## Candidate 7 — PEP 508 markers

### Strengths

- implementation identity is already available;
- conditional dependencies are well established.

### Weakness

A dependency marker answers:

```text
When is dependency X required?
```

It does not answer:

```text
Is distribution X itself supported?
```

There is no normal `Requires-Dist` entry that means “reject this distribution
when its own marker is false”.

### Verdict

**Necessary environment machinery, not a release-support declaration.**

---

## Candidate 8 — PEP 780 ABI features

### Strengths

- handles free-threading and other ABI dimensions;
- prevents implementation name from becoming an overloaded compatibility
  language.

### Weakness

It describes the environment/ABI and dependency applicability, not producer
support policy.

### Verdict

**Complementary and an important scope boundary.**

---

## Candidate 9 — PEP 725 external dependency metadata

### Strengths

- addresses build/host/runtime dependency information;
- explicitly distinguishes build machine and host machine;
- relevant to source-build failures.

### Weakness

It does not currently define Python implementations as the release-support
vocabulary under investigation.

It also separates build dependencies from runtime dependencies, which is
exactly why a build-time CPython requirement should not automatically become a
runtime support declaration.

### Verdict

**Important alternative/complement; not currently equivalent.**

---

## Candidate 10 — source-build policy

Example concept:

```text
Do not automatically build this sdist.
```

### Strengths

- directly targets expensive/failing source builds;
- potentially avoids the immediate operational failure.

### Weakness

It answers:

```text
Should an installer build this source?
```

rather than:

```text
Which Python implementations does the producer support?
```

For a `py3-none-any` wheel, source-build avoidance may not solve a runtime
compatibility mismatch at all.

### Verdict

**Adjacent problem, not equivalent semantics.**

---

## Candidate 11 — no new standard

### Strengths

- zero new metadata burden;
- avoids premature standardization;
- lets tooling combine classifiers, wheel tags and documentation.

### Weakness

The strongest residual cases remain awkward:

```text
py3-none-any
+
CPython-only producer policy
```

There is no standardized normative release-level statement.

### Verdict

**Still a credible final outcome. The research must be willing to choose it.**

---

## Current matrix

| Solution | Semantic fit | Sdist | Resolver | Existing | False-positive risk |
|---|---:|---:|---:|---:|---:|
| Trove classifier | High | Medium | Low | High | Low |
| Reinterpreted classifier | Medium | Medium | High | High | High |
| Requires-Implementation | High | High | High | Low | Medium |
| Supported-Implementation | High | High | High | Low | Medium |
| Wheel tags | High for artifact | Low | High | High | Low |
| PEP 825 variants | High for artifact | Low | High | Emerging | Medium |
| PEP 508 | High for deps | Medium | High for deps | High | Low |
| PEP 780 | High for ABI | Medium | High for ABI/deps | Emerging | Low |
| PEP 725 | High for build/external deps | High | Emerging | Emerging | Medium |
| Source-build policy | Medium | High | High | Low | Medium |
| No new standard | Medium | Medium | Medium | High | Lowest |

This matrix is a research instrument, not a recommendation.
