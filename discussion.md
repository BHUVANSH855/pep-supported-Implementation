# Discussion Record

This file records the externally visible reasoning that should be safe to
reference when discussing the research with packaging maintainers.

## January 2024 implementation-metadata discussion

The thread “Python implementation in metadata” asked whether Python
implementation information should be stored in packaging metadata.

Paul Moore questioned the use case and said that classifiers were appropriate
for declaring which implementations a project is willing to support, without
forcing anything.

C.A.M. Gerlach also pointed to classifiers, wheel tags, and backend/plugin
approaches.

Source:
https://discuss.python.org/t/python-implementation-in-metadata/42653

### Important interpretation

The thread should **not** be summarized as:

> The community rejected implementation metadata.

That is not what happened.

The thread addressed a user's desire to force an implementation in a
development/application context.

The current research asks a narrower question:

> Should a package release have a normative machine-actionable declaration of
> its supported Python implementations for candidate selection?

Those are related but not identical questions.

---

## Free-threading classifier discussion

A later packaging discussion around free-threaded Python is useful prior art.

Participants distinguished:

```text
a free-threaded wheel exists
```

from:

```text
the package actually works correctly without the GIL
```

One proposed role for a classifier was to let the publisher explicitly
signal that users can use the package on a free-threaded build.

Source:
https://discuss.python.org/t/free-threading-trove-classifier/62406

### Why this matters

This is a strong precedent for the distinction:

```text
artifact availability
        ≠
producer support claim
```

It is also a warning against overclaiming what a wheel tag proves.

---

## PEP 725

PEP 725 is directly relevant to build-time and host-time dependency
questions.

It distinguishes:

```text
build machine
host machine
runtime dependencies
```

and proposes standardized external dependency metadata.

Source:
https://peps.python.org/pep-0725/

The research therefore does not claim that PEP 725 is irrelevant.

Instead:

> PEP 725 must be evaluated as an alternative/complement for build-time
> implementation requirements.

Its current scope does not provide the release-level Python implementation
support declaration investigated here.

---

## Current evidence position

The research now has four strong claims:

1. Python implementation identity is already standardized.
2. Built-artifact implementation compatibility is already standardized.
3. Projects can already describe implementation support through classifiers.
4. Some real releases explicitly declare implementation restrictions while
   publishing implementation-generic wheels.

The unresolved question is therefore **semantic and operational**, not whether
the ecosystem has any implementation information at all.

---

## Current strongest cases

- RestrictedPython 8.5
- HAX 0.3.0
- Likepy 0.3.0
- simple-ctx-log 0.0.3
- TribeCore 4.7.3

See `evidence/residual-cases.md`.

---

## Questions for reviewers

A reviewer visiting this repository should be able to answer:

### 1. Is this actually different from classifiers?

If not, a new field may not be justified.

### 2. Is this actually different from wheel tags?

For the strongest cases, the wheel is `py3-none-any` or `py3-none-platform`
while the producer says CPython-only.

### 3. Is this actually different from PEP 508?

Yes if the claim concerns the package itself rather than conditional
dependencies, but this should be demonstrated with concrete resolver
behavior.

### 4. Is this actually different from PEP 780?

Yes: implementation identity and ABI features are separate dimensions.

### 5. Is this actually different from PEP 725?

Potentially: build requirements and release support are not necessarily the
same statement.

### 6. Why Core Metadata?

Because a release-level fact needs to travel with the release and can
potentially be served through existing Core Metadata mechanisms.

But index-level metadata remains a viable alternative.

### 7. What would falsify the proposal?

If existing classifiers plus improved tooling provide an equivalent safe
answer, or if the residual cases have no meaningful consumers, a new field
may not be warranted.

---

## Current repository stance

The repository is intentionally **not a specification**.

It is an evidence-backed pre-PEP research record.

The preferred outcome is whichever conclusion survives review, including:

```text
add a field
```

or:

```text
extend an existing mechanism
```

or:

```text
do nothing new
```
