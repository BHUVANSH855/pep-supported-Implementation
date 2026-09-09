# Requirement vs Supported Implementation

The original proposal used the name `Requires-Implementation`.

The empirical cases and subsequent root-cause analysis make the semantic choice more important than the field name itself.

The central issue is that several different statements can describe implementation compatibility:

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

The research therefore treats the requirement/support distinction as a **semantic question that must be resolved before syntax is designed**.

---

## 1. Requirement

A requirement-style field could look like:

```text
Requires-Implementation: cpython
```

Possible meaning:

> This release requires the CPython implementation and cannot be used with other Python implementations.

This resembles:

```text
Requires-Python: >=3.10
```

where the metadata expresses a condition that must be satisfied by the target environment.

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
* the project intentionally uses private CPython behavior;
* a security property depends on CPython-specific behavior.

These cases have different implications.

In particular:

```text
not supported
    ≠
technically impossible
```

Therefore a `Requires-Implementation` field could accidentally turn a maintainer support policy into a hard resolver constraint.

The key unresolved question is:

> When a producer says "CPython only," is it making a hard compatibility assertion or merely defining its supported product boundary?

---

## 2. Support declaration

A support-style field could look like:

```text
Supported-Implementation: cpython
```

Possible meaning:

> The producer explicitly supports this release on CPython.

This better matches some empirical cases because the producer's actual statement is often about support rather than formal impossibility.

It also avoids automatically interpreting every omitted implementation as technically incompatible.

However, a support declaration introduces its own semantic questions.

The standard would need to define what "supports" means.

For example:

```text
supported
    ≠
tested once
```

and:

```text
supported
    ≠
guaranteed to work under every possible configuration
```

The unresolved question is:

> What producer commitment should a standardized "supported implementation" declaration represent?

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

A producer that lists only CPython would effectively declare every other implementation outside the supported set.

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

The research currently treats **Model B as the safer default hypothesis**, but has not established that it provides enough practical value.

If consumers need to avoid known-incompatible releases, an exhaustive model may be more useful.

If producers cannot reliably maintain exhaustive support lists, an exhaustive model may be unsafe.

This remains a central unresolved design trade-off.

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
* or another precisely defined concept.

Without a precise definition, automated consumers cannot safely interpret the field.

The research should therefore avoid using CI coverage as a substitute for support semantics.

---

## 5. Support is not the same as observed compatibility

A package may happen to work on an implementation that its maintainers do not support.

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

A support metadata field would represent a producer declaration, not independent proof of compatibility.

This creates an important trust problem for any machine-actionable design.

The research should therefore distinguish:

```text
observed compatibility
```

from:

```text
producer-declared support
```

throughout the evidence corpus.

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

PEP 725 is therefore relevant whenever the underlying cause is a build or host requirement.

A source-build failure involving CPython does not, by itself, demonstrate the need for implementation-support metadata.

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

PEP 780 is therefore relevant when the actual restriction is an ABI or environment feature rather than implementation identity.

A possible support declaration should not become a replacement for ABI metadata.

The research should classify such cases as residual implementation-support evidence only when the implementation identity itself is the relevant restriction after ABI/configuration mechanisms have been considered.

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

The existence of such a package does not automatically establish that the packaging ecosystem needs a normative compatibility constraint.

The research therefore needs to distinguish:

```text
private implementation usage
```

from:

```text
general-purpose package support boundary
```

before counting such cases as evidence for a new metadata field.

The simple-ctx-log evidence is useful here because its release documentation explicitly identifies CPython-only behavior while its source uses `sys._getframe`.

That strengthens the evidence for an implementation-specific support boundary, but does not by itself prove that a new metadata mechanism is required.

---

## 9. Support is not the same as alternative-interpreter failure

A package may fail on PyPy because PyPy currently lacks a feature or has a bug.

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

A temporary ecosystem compatibility problem should not automatically become a permanent package metadata restriction.

This is why root-cause classification is required before a real-world case is treated as residual evidence.

The research should ask:

> Is the producer declaring a durable support boundary, or is the package merely encountering a current implementation defect?

---

## 10. Can security support boundaries be represented by ordinary compatibility semantics?

Some projects have implementation restrictions for reasons stronger than ordinary portability.

RestrictedPython is an important example because its project documentation and source distinguish CPython from other implementations in the context of its security guarantees.

This raises a separate semantic question:

> Is a security-related support boundary different from an ordinary compatibility boundary?

If a producer says that use on another implementation could undermine a security property, an informational classifier may be insufficient.

However, this still does not automatically imply that a `Requires-Implementation` field is correct.

The research must determine whether such cases require:

* hard compatibility semantics;
* advisory support metadata;
* another security-specific mechanism;
* or no normative packaging mechanism.

---

## 11. What do the empirical cases actually demonstrate?

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

HAX 0.3.0 is currently the strongest runtime candidate because its release source explicitly enforces a CPython-only runtime condition.

Other cases, including RestrictedPython, simple-ctx-log, Likepy, and TribeCore, remain useful but differ in root cause and evidence strength.

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

The purpose of this document is to preserve that distinction.

---

## 12. Why `Requires-Implementation` is attractive

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

This is attractive if the producer's statement is genuinely a hard compatibility constraint.

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

That may be inappropriate for:

* support-policy declarations;
* incomplete testing;
* temporary alternative-interpreter failures;
* optional functionality;
* environments where the package happens to work despite lack of official support.

Requirement semantics therefore need stronger evidence than simply finding a project that says "CPython only."

---

## 13. Why `Supported-Implementation` is attractive

A support model better reflects the language used by many projects:

```text
CPython supported
PyPy unsupported
```

rather than a formal statement:

```text
PyPy is technically impossible.
```

It could naturally represent multiple implementations:

```text
Supported-Implementation:
    cpython
    pypy
    graalpy
```

The model is therefore semantically closer to a producer's support relationship.

### Main risk

The word "supported" is inherently less precise than "required".

Without a normative definition, different producers could use it to mean:

```text
tested
known working
officially supported
best effort
expected to work
```

A standard would need to eliminate that ambiguity.

There is also a practical question:

> If the field does not cause candidate rejection, what concrete consumer value does it provide beyond existing classifiers and project documentation?

That question remains unresolved.

---

## 14. Possible semantic models

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

This model requires strong evidence that the producer is asserting a true compatibility boundary rather than a support policy.

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

This model is particularly sensitive to stale or incomplete declarations.

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

This is currently the leading semantic hypothesis if a support field is ultimately justified.

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

This model must therefore be evaluated against the possibility of simply improving classifier usage or tooling instead.

---

## 15. What should omission mean?

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

This means an old distribution without the field does not suddenly become declared universally supported or unsupported.

It also preserves a distinction between:

```text
explicitly supported
```

and:

```text
unknown
```

This is particularly important if the field is introduced after a large existing ecosystem already exists.

---

## 16. What should an unlisted implementation mean?

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

This is one of the most important unresolved questions because it determines how much candidate filtering the metadata can provide.

The research should not assume that "not listed" and "known incompatible" are equivalent.

---

## 17. How should multiple declarations work?

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

The likely simple interpretation would be:

```text
current implementation ∈ declared support set
```

but this has not been standardized.

Conditional support creates additional complexity.

For example:

```text
CPython:
    supported

PyPy:
    supported only when optional dependency X exists
```

A simple implementation list cannot necessarily express such conditions.

The research should resist turning the field into a general-purpose compatibility expression language.

---

## 18. How should implementation identity be matched?

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

The research should avoid creating a second incompatible closed registry if possible.

Existing implementation classifiers also provide an ecosystem vocabulary, although their current semantics are descriptive rather than normative candidate constraints.

The unresolved question is:

> Should a future support declaration reuse an existing implementation-name vocabulary, and if so, which source should define its canonical values?

---

## 19. How should Python versions interact with implementation support?

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

A package could also have different implementation/version relationships:

```text
CPython:
    supported for 3.10+

PyPy:
    supported for 3.11+
```

If the field cannot express such conditions cleanly, the standard should not attempt to duplicate Python version semantics.

This raises a deeper question:

> Is implementation support sufficiently independent from Python version support to justify a separate metadata dimension?

---

## 20. How should ABI features interact with support?

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

This prevents `Supported-Implementation` from becoming an overloaded compatibility language.

---

## 21. Could classifiers acquire normative semantics instead?

Another possibility is to retain the existing classifier vocabulary and define additional semantics for implementation classifiers.

For example:

```text
Programming Language :: Python :: Implementation :: CPython
```

could potentially be interpreted by tools as support information.

However, changing the semantics of existing classifiers creates compatibility and ecosystem-adoption questions.

There is also empirical evidence that existing classifiers are not currently treated as hard candidate constraints.

A controlled package containing:

```text
CPython implementation classifier
+
Requires-Python >=3.8
+
py3-none-any wheel
```

was still selected by the tested pip path under a simulated PyPy target.

A corresponding tested uv path also resolved the generic artifact, although that test interface did not explicitly simulate a PyPy implementation identity.

Therefore the experiment supports the narrower conclusion that **existing implementation classifiers are not presently equivalent to normative implementation compatibility constraints in the tested packaging paths**.

It does not establish that the ecosystem needs a new field.

The research therefore needs to compare:

```text
new normative field
```

against:

```text
normative interpretation of existing classifiers
```

and against:

```text
better classifier/tooling conventions
```

rather than assuming that a new field is the only route to machine actionability.

---

## 22. What would a resolver do?

A support field is only useful for candidate selection if its semantics map to a concrete resolver decision.

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

The stronger unresolved question is:

> What specific resolver behavior would produce enough benefit to justify introducing this information into candidate selection?

If the answer is "none," the case for normative resolver semantics weakens substantially.

---

## 23. What happens when the declaration is wrong?

Suppose a release says:

```text
Supported-Implementation: cpython
```

but later works perfectly on PyPy.

Possible consequence:

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

This trust problem is a major argument against making support metadata an unconditional hard requirement without further evidence.

The unresolved question is:

> Can stale or incorrect declarations be detected or mitigated well enough that automated consumers can safely depend on them?

---

## 24. How should release-level support interact with artifact-level compatibility?

Wheel tags describe individual artifacts.

A release-level support declaration would describe a producer's support relationship with the release.

These are different layers:

```text
artifact:
    "this wheel is compatible with environment X"

release:
    "this producer supports this implementation"
```

A release may contain:

```text
generic pure-Python wheel
+
platform-specific wheel
+
sdist
```

The artifacts can therefore have different compatibility properties even when they belong to the same release.

The unresolved question is:

> Can a single release-level implementation-support declaration accurately coexist with heterogeneous artifacts?

If artifact metadata already communicates everything required for the relevant consumer decision, a separate release-level declaration may not be necessary.

---

## 25. How should source distributions affect the distinction?

The release-level hypothesis becomes most interesting when an sdist exists without an already-compatible wheel.

For example:

```text
sdist
    ↓
installer considers source build
    ↓
build executes under current implementation
    ↓
failure
```

A release-level support declaration could theoretically allow a consumer to avoid the build.

But the research must establish that the failure is actually a runtime/support boundary rather than:

* a build dependency problem;
* a host requirement;
* an external dependency;
* a toolchain problem;
* an ABI issue;
* a temporary interpreter bug;
* or an artifact publication problem.

The presence of an sdist is therefore not itself evidence for requirement semantics.

---

## 26. Could failed-build caching change the requirement/support analysis?

A resolver or installer could potentially record:

```text
release R
+
environment X
+
build attempt
+
failure
```

and avoid repeating the same work.

This provides a fundamentally different type of information:

```text
observed failure
```

rather than:

```text
producer support declaration
```

Caching may therefore address some practical source-build costs without defining a new metadata semantics.

However:

* the first attempt still occurs;
* the result may depend on the exact environment;
* failures may be transient;
* cached results may become stale;
* one environment's failure may not imply another environment's failure.

The unresolved question is:

> Is the primary practical problem actually semantic uncertainty, or is it repeated failed work that could be addressed by caching?

This distinction should be resolved before claiming that metadata is necessary.

---

## 27. What would a non-installer consumer do?

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

> Which concrete consumer has the strongest need, and can that need justify standardization independently of speculative future consumers?

---

## 28. What would justify requirement semantics?

A requirement-style field becomes more compelling if the research finds releases where:

1. the producer explicitly establishes a hard implementation boundary;
2. the boundary is technically meaningful rather than merely a support policy;
3. existing artifact metadata cannot represent it;
4. existing build metadata does not adequately represent it;
5. the consumer must avoid the candidate before installation/build;
6. treating the boundary as advisory would provide insufficient protection;
7. the declaration can be obtained before the operation it is intended to avoid.

If those conditions are repeatedly demonstrated, `Requires-Implementation` could become a defensible design.

The important point is that **"CPython only" alone is not sufficient evidence for all seven conditions**.

---

## 29. What would justify support semantics?

A support-style field becomes more compelling if the research finds that:

1. producers routinely need to communicate implementation support;
2. the information is useful to consumers before installation;
3. the support relationship cannot be adequately represented by classifiers;
4. hard incompatibility semantics are too strong;
5. positive support information still provides meaningful consumer value;
6. producers can maintain declarations accurately enough;
7. consumers have a defined action to take with positive support information.

This would favor a field such as:

```text
Supported-Implementation
```

with carefully defined positive semantics.

However, if the only meaningful consumer action is hard rejection of unlisted implementations, a positive-only support model may not solve the original problem.

---

## 30. What would justify informational semantics?

An informational implementation-support field becomes more compelling if the primary benefits are:

* ecosystem discovery;
* test-matrix generation;
* compatibility analysis;
* package-index presentation;
* maintainer communication;
* quality tooling.

This model has lower risk because it does not change candidate validity.

However, the burden of adding a new Core Metadata field is difficult to justify if the same information can be represented by existing classifiers.

The unresolved question is:

> What additional semantic precision or machine-readable value would a new informational field provide that classifiers cannot?

---

## 31. Current terminology hypothesis

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

> `Supported-Implementation` is currently a design hypothesis, not a final recommendation.

The field name should not be frozen until the semantics are established.

It is also possible that neither name is appropriate if the final research shows that the desired concept is:

* compatibility;
* implementation targeting;
* support policy;
* or another narrower concept.

---

## 32. Current preferred semantic direction

If a new mechanism is eventually justified, the current research direction is to investigate a **positive support declaration** first.

Conceptually:

```text
Supported-Implementation:
    cpython
    pypy
```

would mean:

> These implementations are explicitly supported by the producer for this release.

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

and avoids turning the absence of a declaration into a negative compatibility claim.

But it may also provide insufficient candidate filtering for the use case that motivated the research.

That trade-off remains unresolved.

---

## 33. Current decision gate

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
5. compare artifact-level alternatives
        ↓
6. compare build/host mechanisms
        ↓
7. determine whether hard rejection is required
        ↓
8. determine whether positive support is sufficient
        ↓
9. define omission semantics
        ↓
10. define implementation identity
        ↓
11. evaluate declaration trustworthiness
        ↓
12. compare Core Metadata with alternatives
        ↓
13. choose field semantics/name
```

This prevents the research from becoming an argument over terminology before the underlying problem has been established.

---

# Current conclusion

The empirical evidence makes `Requires-Implementation` less obviously appropriate than the original proposal suggested.

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

as the more promising semantic direction **if** a new mechanism is eventually shown to be necessary.

But the research has not yet established that conclusion.

The strongest current evidence establishes:

```text
some releases have implementation-specific support boundaries
that are not represented by their generic wheel tags or
Requires-Python.
```

It does not yet establish:

```text
therefore a new Core Metadata field is necessary.
```

The central unresolved question remains:

> Is there a sufficiently important class of release-level implementation support boundaries for which existing metadata is inadequate and for which machine-readable support information provides enough consumer value to justify a new normative standard?

Until that question is answered, neither `Requires-Implementation` nor `Supported-Implementation` should be treated as the final design.
