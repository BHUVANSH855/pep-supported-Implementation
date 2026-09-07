# Requirement vs Supported Implementation

The original proposal used the name `Requires-Implementation`.

The empirical cases and subsequent root-cause analysis make the semantic choice
more important than the field name itself.

The central issue is that several different statements can describe
implementation compatibility:

```text
requirement
    =
what the environment must provide

support declaration
    =
what the producer explicitly supports

observed compatibility
    =
what has been observed to work

support policy
    =
what the producer chooses to support
```

These concepts overlap, but they are not equivalent.

---

## 1. Requirement

A requirement-style field could look like:

```text
Requires-Implementation: cpython
```

Possible meaning:

> This release requires the CPython implementation and cannot be used with
> other Python implementations.

This resembles:

```text
Requires-Python: >=3.10
```

where the metadata expresses a condition that must be satisfied by the target
environment.

### Problem

A technical impossibility claim is stronger than a support-policy claim.

A project may say:

```text
CPython only
```

because:

* it has only tested CPython;
* maintainers only provide support for CPython;
* it depends on CPython-specific APIs;
* an alternative implementation currently has a compatibility bug;
* the build process only works on CPython;
* the project intentionally uses private CPython behavior.

These cases have different implications.

In particular:

```text
not supported
    ≠
technically impossible
```

Therefore a `Requires-Implementation` field could accidentally turn a
maintainer support policy into a hard resolver constraint.

---

## 2. Support declaration

A support-style field could look like:

```text
Supported-Implementation: cpython
```

Possible meaning:

> The producer explicitly supports this release on CPython.

This better matches some of the empirical cases because the producer's actual
statement is often about support rather than a formal impossibility claim.

It also avoids automatically interpreting every omitted implementation as
technically incompatible.

However, a support declaration introduces its own semantic questions.

---

## 3. Known support vs exhaustive support

There are at least two fundamentally different interpretations of:

```text
Supported-Implementation: cpython
```

### Model A — exhaustive support set

Only the listed implementations are supported.

Therefore:

```text
CPython
    → supported

PyPy
    → unsupported
```

This gives useful candidate filtering.

But it is also a strong claim.

A producer that lists only CPython would effectively declare every other
implementation outside the supported set.

### Model B — positive support declaration

The listed implementations are explicitly supported.

Unlisted implementations remain unknown.

Therefore:

```text
CPython
    → explicitly supported

PyPy
    → no support declaration
```

This is safer because it does not transform omission into a negative claim.

However, it provides much less candidate filtering.

### Current research position

The research currently treats **Model B as the safer default hypothesis**, but
has not established that it provides enough practical value.

If consumers need to avoid known-incompatible releases, an exhaustive model may
be more useful.

If producers cannot reliably maintain exhaustive support lists, an exhaustive
model may be unsafe.

This is a central unresolved design trade-off.

---

## 4. Support is not the same as testing

Consider:

```text
Supported-Implementation: cpython
```

This should not automatically mean:

```text
CPython was tested on every supported Python version,
platform, ABI configuration, and build mode.
```

Likewise:

```text
not listed
```

should not automatically mean:

```text
known broken
```

A standard must decide whether the field represents:

* maintainer-supported environments;
* known-working environments;
* tested environments;
* guaranteed environments;
* or some other defined concept.

Without a precise definition, automated consumers cannot safely interpret the
field.

---

## 5. Support is not the same as observed compatibility

A package may happen to work on an implementation that its maintainers do not
support.

For example:

```text
PyPy:
    works in an experiment
    but is not supported by the project
```

Conversely:

```text
PyPy:
    is listed as supported
    but a particular release contains a bug
```

Therefore:

```text
observed to work
    ≠
supported
```

A support metadata field would represent a producer declaration, not an
independent proof of compatibility.

This creates an important trust problem for any machine-actionable design.

---

## 6. Support is not the same as a build requirement

Consider:

```text
The package can only run on CPython.
```

versus:

```text
The package's build process must execute under CPython.
```

The first is a runtime support statement.

The second may instead be caused by:

* build dependencies;
* host dependencies;
* compiler/toolchain requirements;
* external libraries;
* build-system behavior;
* alternative-interpreter build limitations.

These should not automatically be represented by the same field.

PEP 725 is therefore relevant whenever the underlying cause is a build or host
requirement.

---

## 7. Support is not the same as ABI compatibility

A package can support:

```text
CPython
```

but only under particular ABI/configuration conditions.

For example:

```text
CPython + regular ABI
    → supported

CPython + free-threaded ABI
    → unsupported
```

or:

```text
CPython + non-debug
    → supported

CPython + debug build
    → unsupported
```

These distinctions belong to a different compatibility dimension.

PEP 780 is therefore relevant when the actual restriction is an ABI feature
rather than implementation identity.

A possible support declaration should not become a replacement for ABI
metadata.

---

## 8. Support is not the same as private implementation usage

A project may intentionally use CPython-specific internals.

For example:

```text
CPython private API
        ↓
CPython-specific package
```

This may result in:

```text
CPython:
    supported

other implementations:
    outside project scope
```

The existence of such a package does not automatically establish that the
packaging ecosystem needs a normative compatibility constraint.

The research therefore needs to distinguish:

```text
private implementation usage
```

from:

```text
general-purpose package support boundary
```

before counting such cases as evidence for a new metadata field.

---

## 9. Support is not the same as alternative-interpreter failure

A package may fail on PyPy because PyPy currently lacks a feature or has a
bug.

For example:

```text
package
    ↓
uses feature X
    ↓
alternative implementation lacks/fails X
```

There are at least two possible responses:

```text
package metadata:
    declare incompatibility
```

or:

```text
interpreter:
    fix compatibility problem
```

A temporary ecosystem compatibility problem should not automatically become a
permanent package metadata restriction.

This is why root-cause classification is required before a real-world case is
treated as residual evidence.

---

## 10. What do the empirical cases actually demonstrate?

The strongest current cases demonstrate statements of the form:

```text
producer:

    CPython only

artifact:

    py3-...

sdist:

    present

Requires-Python:

    version restriction only
```

HAX 0.3.0 is currently the strongest candidate because the CPython restriction
is explicitly enforced at runtime.

Other cases, including RestrictedPython, Likepy, simple-ctx-log, and TribeCore,
remain useful but require more root-cause validation.

These cases establish a potentially important distinction:

```text
artifact compatibility
        ≠
producer-declared implementation support
```

They do **not** yet establish:

```text
therefore Requires-Implementation is necessary
```

or:

```text
therefore Supported-Implementation is necessary
```

---

## 11. Why `Requires-Implementation` is attractive

The requirement model has an intuitive relationship with existing metadata:

```text
Requires-Python
Requires-Implementation
```

A resolver could conceptually evaluate:

```text
Python version
+
Python implementation
```

before selecting a candidate.

This is attractive if the producer's statement is genuinely a hard
compatibility constraint.

It would be particularly useful if:

```text
candidate matches Python version
+
candidate artifact appears generic
+
producer knows it cannot work on current implementation
```

and the resolver could eliminate the candidate before a source build.

### Main risk

The model encourages a strong interpretation:

```text
not satisfying Requires-Implementation
    =
candidate is invalid
```

That may be inappropriate for support-policy declarations or incomplete
testing.

---

## 12. Why `Supported-Implementation` is attractive

A support model better reflects the language used by many projects:

```text
CPython supported
PyPy unsupported
```

rather than a formal statement:

```text
PyPy is technically impossible.
```

It could also naturally represent multiple implementations:

```text
Supported-Implementation:
    cpython
    pypy
    graalpy
```

The model is therefore semantically closer to the producer's support
relationship.

### Main risk

The word “supported” is inherently less precise than “required”.

Without a normative definition, different producers could use it to mean:

```text
tested
known working
officially supported
best effort
expected to work
```

A standard would need to eliminate that ambiguity.

---

## 13. Possible semantic models

The research can therefore be represented as four broad models.

### Model 1 — hard requirement

```text
Requires-Implementation: cpython
```

Semantics:

```text
current implementation must be CPython
```

Consumer behavior:

```text
non-CPython → reject candidate
```

Strength:

```text
clear candidate-selection semantics
```

Risk:

```text
turns support statements into hard incompatibility claims
```

---

### Model 2 — exhaustive support declaration

```text
Supported-Implementation: cpython
```

Semantics:

```text
only listed implementations are supported
```

Consumer behavior:

```text
non-listed → reject/deprioritize
```

Strength:

```text
useful candidate filtering
```

Risk:

```text
omission becomes a strong negative claim
```

---

### Model 3 — positive support declaration

```text
Supported-Implementation: cpython
```

Semantics:

```text
CPython is explicitly supported
other implementations are unknown
```

Consumer behavior:

```text
CPython → positive information
other implementations → insufficient information
```

Strength:

```text
conservative and backwards-compatible
```

Risk:

```text
limited ability to avoid known-incompatible candidates
```

---

### Model 4 — informational implementation targeting

```text
Implementation: cpython
```

Semantics:

```text
descriptive producer information
```

Consumer behavior:

```text
no normative candidate rejection
```

Strength:

```text
low compatibility risk
```

Risk:

```text
may provide little beyond existing classifiers
```

---

## 14. What should omission mean?

This question is inseparable from the requirement/support distinction.

For a requirement field:

```text
absence
```

could reasonably mean:

```text
no additional requirement
```

For a support field, that interpretation is dangerous.

The current preferred interpretation is:

```text
absence
    =
no normative implementation-support declaration
```

This means an old distribution without the field does not suddenly become
declared universally supported or unsupported.

It also preserves a distinction between:

```text
explicitly supported
```

and:

```text
unknown
```

---

## 15. What should an unlisted implementation mean?

This depends on the chosen model.

Under an exhaustive model:

```text
Supported-Implementation: cpython
```

means:

```text
PyPy → unsupported
```

Under a positive model:

```text
Supported-Implementation: cpython
```

means:

```text
PyPy → unknown
```

This is one of the most important unresolved questions because it determines
how much candidate filtering the metadata can provide.

---

## 16. How should multiple declarations work?

A release may support multiple implementations:

```text
Supported-Implementation:
    cpython
    pypy
```

The semantics must clarify whether this means:

```text
OR
```

or something more complicated.

The likely intended interpretation would be:

```text
current implementation ∈ declared support set
```

but this has not yet been standardized.

Conditional support creates additional complexity.

For example:

```text
CPython:
    supported

PyPy:
    supported only when optional dependency X exists
```

A simple implementation list cannot necessarily express such conditions.

The research should resist turning the field into a general-purpose
compatibility expression language.

---

## 17. How should implementation identity be matched?

A standard must define what counts as an implementation identity.

Possible examples include:

```text
cpython
pypy
graalpy
```

But implementation ecosystems can contain:

* forks;
* derivatives;
* compatibility layers;
* new implementations;
* implementation families.

PEP 421's approach to implementation identity is relevant here.

The research should avoid creating a second incompatible closed registry if
possible.

---

## 18. How should Python versions interact with implementation support?

Implementation support cannot replace `Requires-Python`.

For example:

```text
Supported-Implementation: cpython
Requires-Python: >=3.10,<3.16
```

could jointly describe:

```text
implementation = CPython
version ∈ [3.10, 3.16)
```

The two dimensions should remain independent.

A package could also have:

```text
CPython:
    supported for 3.10+

PyPy:
    supported for 3.11+
```

If the field cannot express such conditions cleanly, the standard should not
attempt to duplicate Python version semantics.

---

## 19. How should ABI features interact with support?

Likewise:

```text
implementation identity
```

should remain separate from:

```text
ABI/configuration
```

A conceptual compatibility evaluation might therefore look like:

```text
implementation
+
Python version
+
ABI features
+
platform
+
artifact tags
```

Each dimension should have a clearly defined owner.

This prevents `Supported-Implementation` from becoming an overloaded
compatibility language.

---

## 20. Could classifiers acquire normative semantics instead?

Another possibility is to retain the existing classifier vocabulary and define
additional semantics for implementation classifiers.

For example:

```text
Programming Language :: Python :: Implementation :: CPython
```

could potentially be interpreted by tools as support information.

However, changing the semantics of existing classifiers creates compatibility
and ecosystem-adoption questions.

The research therefore needs to compare:

```text
new normative field
```

against:

```text
normative interpretation of existing classifiers
```

rather than assuming that a new field is the only way to obtain machine
actionability.

---

## 21. What would a resolver do?

A support field is only useful for candidate selection if its semantics map to
a concrete resolver decision.

Possible behaviors include:

```text
declared supported
    → normal candidate

declared unsupported
    → reject

declared unsupported
    → deprioritize

not declared
    → normal candidate

not declared
    → warn
```

The research currently does not select one.

A particularly important requirement is that:

```text
unknown
```

should not accidentally become:

```text
unsupported
```

unless the final semantics explicitly choose an exhaustive support model.

---

## 22. What happens when the declaration is wrong?

Suppose a release says:

```text
Supported-Implementation: cpython
```

but later works perfectly on PyPy.

Possible consequences include:

```text
false negative
    ↓
installer rejects a usable candidate
```

The reverse is also possible:

```text
Supported-Implementation: cpython, pypy
```

when PyPy support is actually broken.

Possible mitigation mechanisms include:

* informational semantics;
* warning-only behavior;
* conservative candidate filtering;
* CI validation;
* package-quality tooling;
* repository checks.

This trust problem is a major argument against making support metadata an
unconditional hard requirement without further evidence.

---

## 23. Current terminology hypothesis

The research currently considers the following terminology more promising:

```text
Requires-Implementation
        ↓
requirement semantics
        ↓
potentially too strong
```

versus:

```text
Supported-Implementation
        ↓
producer support semantics
        ↓
closer to empirical evidence
```

However:

> `Supported-Implementation` is currently a design hypothesis, not a final
> recommendation.

The field name should not be frozen until the semantics are established.

---

## 24. Current preferred semantic direction

If a new mechanism is eventually justified, the current research direction is
to investigate a **positive support declaration** first.

Conceptually:

```text
Supported-Implementation:
    cpython
    pypy
```

would mean:

> These implementations are explicitly supported by the producer for this
> release.

Under this model:

```text
listed implementation
    → positive support declaration

unlisted implementation
    → unknown unless the specification says otherwise
```

This is more conservative than:

```text
unlisted implementation
    → known incompatible
```

and avoids turning the absence of a declaration into a negative compatibility
claim.

But it may also provide insufficient candidate filtering for the very use case
that motivated the research.

That trade-off remains unresolved.

---

## 25. What would justify requirement semantics?

A requirement-style field becomes more compelling if the research finds releases
where:

1. the producer explicitly establishes a hard implementation boundary;
2. the boundary is technically meaningful rather than merely a support policy;
3. existing artifact metadata cannot represent it;
4. existing build metadata does not adequately represent it;
5. the consumer must avoid the candidate before installation/build;
6. treating the boundary as advisory would provide insufficient protection.

If those conditions are repeatedly demonstrated, `Requires-Implementation`
could become a defensible design.

---

## 26. What would justify support semantics?

A support-style field becomes more compelling if the research finds that:

1. producers routinely need to communicate implementation support;
2. the information is useful to consumers before installation;
3. the support relationship cannot be adequately represented by classifiers;
4. hard incompatibility semantics are too strong;
5. positive support information still provides meaningful consumer value;
6. producers can maintain the declarations accurately enough.

This would favor a field such as:

```text
Supported-Implementation
```

with carefully defined positive semantics.

---

## 27. Current decision gate

The research should not decide between:

```text
Requires-Implementation
```

and:

```text
Supported-Implementation
```

yet.

The correct order is:

```text
1. establish real residual cases
        ↓
2. classify root causes
        ↓
3. identify actual consumer decisions
        ↓
4. test existing mechanisms
        ↓
5. determine whether hard rejection is required
        ↓
6. determine whether positive support is sufficient
        ↓
7. define omission semantics
        ↓
8. define implementation identity
        ↓
9. compare Core Metadata with alternatives
        ↓
10. choose field semantics/name
```

This prevents the research from becoming an argument over terminology before
the underlying problem has been established.

---

# Current conclusion

The empirical evidence makes `Requires-Implementation` less obviously
appropriate than the original proposal suggested.

A requirement says:

```text
the environment MUST provide X
```

while a support declaration can say:

```text
the producer explicitly supports X
```

Those are different claims.

The current research therefore treats:

```text
Supported-Implementation
```

as the more promising semantic direction **if** a new mechanism is eventually
shown to be necessary.

But the research has not yet established that conclusion.

The central unresolved question remains:

> Is there a sufficiently important class of release-level implementation
> support boundaries for which existing metadata is inadequate and for which
> machine-readable support information provides enough consumer value to
> justify a new normative standard?

Until that question is answered, neither `Requires-Implementation` nor
`Supported-Implementation` should be treated as the final design.
