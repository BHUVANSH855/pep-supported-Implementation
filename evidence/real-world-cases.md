# Real-World Cases

This document records concrete examples and ecosystem observations relevant to implementation-level compatibility.

It is intentionally broader than `residual-cases.md`.

A real-world case demonstrates that a particular behavior, restriction, workflow, or metadata mismatch exists. It does **not** automatically demonstrate that:

* the behavior requires new metadata;
* the metadata should be normative;
* installers should act on it;
* the information belongs in Core Metadata;
* or an implementation-support field is preferable to existing mechanisms.

The research therefore distinguishes:

```text
real-world observation
        ↓
root-cause classification
        ↓
residual case
        ↓
standardization requirement
```

Only the latter stages can support a proposal for new normative metadata.

---

## 1. HAX 0.3.0

HAX 0.3.0 is currently the strongest runtime-oriented implementation-support case in the corpus.

The release publishes a generic Python wheel alongside a source distribution, while its source contains an explicit runtime implementation check:

```python
if implementation.name != "cpython":
    raise RuntimeError("HAX only supports CPython!")
```

The project description likewise states that HAX supports CPython 3.7+.

The important combination is therefore:

```text
producer:
    CPython only

artifact:
    py3-none-any

sdist:
    present

Requires-Python:
    Python-version constraint

runtime:
    explicit CPython-only guard
```

### Why this matters

This is a stronger case than merely observing an implementation classifier.

The release contains an operational boundary:

```text
current implementation != CPython
        ↓
runtime failure
```

while the wheel's Python tag is generic.

This demonstrates that:

```text
wheel compatibility
        ≠
complete producer support information
```

at least for this release.

### Root-cause interpretation

The current classification is primarily:

```text
A — runtime semantic restriction
```

with the producer's explicit support policy reinforcing the classification.

The restriction is not merely inferred from the absence of PyPy testing.

### What remains unproven

HAX does not by itself prove that:

* pip should reject the release on PyPy;
* the generic wheel is incorrectly tagged;
* `Requires-Implementation` is the correct solution;
* `Supported-Implementation` is the correct solution;
* the information must be stored in Core Metadata;
* or a resolver would materially benefit from knowing this before installation.

The remaining question is whether the release creates a concrete pre-install decision that existing mechanisms cannot make.

### Evidence status

**Strong runtime candidate; not yet independently sufficient as residual evidence.**

---

## 2. simple-ctx-log 0.0.3

simple-ctx-log 0.0.3 is an implementation-specific case involving low-level CPython behavior.

The project describes the use of `sys._getframe` as CPython-only behavior.

The released source contains:

```python
def _find_caller_frame(self) -> FrameType | None:
    try:
        frame = sys._getframe(2)
    except ValueError:
        return None
```

The release publishes a generic `py3-none-any` wheel and an sdist.

### Why this matters

The case combines:

```text
generic artifact
+
CPython-specific low-level API
+
explicit implementation boundary
```

This is relevant because a pure-Python wheel can still have a narrower implementation-support boundary than its artifact tag suggests.

### Root-cause interpretation

The current classification is primarily:

```text
E — private implementation usage
F — explicit support-policy declaration
```

The use of an implementation-specific API provides a concrete technical reason for the producer's support boundary.

However, the research has not established that every use of a private CPython API should become a normative installation constraint.

### What remains unproven

The case does not establish:

* that PyPy necessarily fails for every relevant operation;
* that the project could not provide a fallback;
* that installers should reject PyPy;
* that classifiers are insufficient for the project's intended communication;
* or that a new Core Metadata field is necessary.

### Evidence status

**Strong implementation-specific support candidate; residuality still requires consumer-benefit analysis.**

---

## 3. RestrictedPython 8.5

RestrictedPython provides a different kind of implementation boundary.

Its current project documentation explicitly limits support to CPython and explains that its restrictions cannot be provided safely on other Python implementations.

The source also identifies CPython-specific behavior and warns when used with another implementation.

This is important because the implementation boundary is not merely a consequence of missing tests.

### Why this matters

The case demonstrates that a producer may have an implementation-specific support boundary because the project's intended guarantees depend on interpreter behavior.

Conceptually:

```text
implementation identity
        ↓
security/restriction semantics
        ↓
producer support boundary
```

This is stronger than:

```text
implementation identity
        ↓
untested environment
```

### Root-cause interpretation

The current classification is primarily:

```text
F — explicit support-policy declaration
```

with implementation-specific technical assumptions underlying the policy.

### Important qualification

This case should not automatically be treated as evidence for ordinary package compatibility metadata.

Security-sensitive guarantees may require different treatment from ordinary portability.

The unresolved question is whether the packaging ecosystem needs a generic implementation-support field for such cases, or whether security-sensitive projects should communicate their restrictions through another mechanism.

### Evidence status

**Strong support-policy case and important semantic boundary case; not automatically a resolver residual.**

---

## 4. Likepy 0.3.0

Likepy 0.3.0 explicitly states that it supports CPython and publishes a generic `py3-none-any` wheel together with an sdist.

It also carries a CPython implementation classifier.

The release therefore provides another example of:

```text
producer:
    CPython only

artifact:
    implementation-generic

classifier:
    CPython
```

### Why this matters

Likepy demonstrates the coexistence of three different information layers:

```text
producer documentation
+
Trove classifier
+
generic wheel compatibility
```

This is useful for testing whether the existing descriptive mechanisms already provide enough information for human consumers.

### Root-cause interpretation

The root cause has not yet been established sufficiently to classify this as a confirmed runtime residual.

Possible explanations include:

* implementation-specific runtime behavior;
* private API usage;
* support policy;
* dependency behavior;
* or another project-specific restriction.

The source repository was not reliably retrieved during the current investigation.

### Important qualification

Likepy should therefore **not** be promoted to strong residual evidence merely because its documentation says "CPython only."

The missing step is:

```text
explicit support statement
        ↓
source/root-cause validation
        ↓
consumer decision
```

### Evidence status

**Candidate; root cause and residuality unresolved.**

---

## 5. TribeCore 4.7.3

TribeCore 4.7.3 provides a useful counterexample to simplistic interpretation of platform-specific wheels.

The project publishes wheels such as:

```text
py3-none-win_amd64
py3-none-manylinux2014_x86_64.manylinux_2_17_x86_64
py3-none-macosx_13_0_universal2
```

The project describes its native functionality as a Zig library accessed through `ctypes` and states support for Python 3.10–3.13.

### Why this matters

The artifacts are platform-specific while remaining Python-implementation-generic.

This demonstrates that:

```text
native functionality
        ≠
automatically CPython-only ABI
```

A native component can sometimes avoid coupling itself to the CPython extension ABI.

### Root-cause interpretation

The current classification is primarily:

```text
F — explicit support-policy declaration
```

rather than a confirmed implementation-ABI restriction.

### Important qualification

TribeCore should not be used as evidence that a new implementation metadata field is required.

Its current evidence is better interpreted as a control case showing why the research must distinguish:

```text
native dependency
```

from:

```text
CPython-specific ABI requirement
```

### Evidence status

**Control/support-policy case; not a confirmed residual.**

---

## 6. Guppy3

Guppy3 is an important implementation/ABI boundary case.

Its build configuration explicitly checks:

```python
sys.implementation.name != "cpython"
```

The project's published information also distinguishes:

* CPython support;
* unsupported PyPy and other implementations;
* unsupported free-threaded CPython.

### Why this matters

Guppy3 demonstrates that a package can have multiple compatibility dimensions:

```text
Python implementation identity
        +
ABI/configuration characteristics
        +
Python version
        +
build/artifact compatibility
```

This makes it useful for investigating where implementation identity fits relative to existing packaging mechanisms.

It also demonstrates why an implementation-support mechanism cannot be treated as a universal replacement for version, ABI, or artifact metadata.

### Root-cause interpretation

The current evidence places Guppy3 primarily near:

```text
D — ABI/configuration restriction
E — private/implementation-specific usage
```

with build-time implementation checks also present.

### What it does not prove

Guppy3 does not prove that:

* every CPython-only project needs metadata;
* installers must reject PyPy;
* a new Core Metadata field is required;
* PEP 725 cannot solve the build-time part of the problem;
* implementation identity alone is sufficient to describe compatibility.

Guppy3 should therefore be treated as an important **boundary/control case**, not the cleanest residual case.

### Evidence status

**Boundary/control case.**

---

## 7. Specialist

The `brandtbucher/specialist` project provides another useful example of an explicit implementation restriction.

Its runtime behavior includes an explicit message equivalent to:

```text
Specialist only supports CPython 3.11+!
```

### Why this matters

This is useful because the restriction combines:

```text
implementation
+
Python version
```

rather than implementation identity alone.

It therefore tests whether a proposed implementation field can coexist cleanly with `Requires-Python`.

### Root-cause interpretation

The implementation restriction is explicit, but the broader root cause and artifact-level behavior require further validation before treating it as residual evidence.

### Research implication

A statement such as:

```text
CPython 3.11+
```

could potentially be represented as the intersection of:

```text
implementation identity
+
Python version
```

If the implementation field requires duplicating version semantics, that would be a design warning.

### Evidence status

**Useful candidate for cross-dimension analysis; not yet promoted to confirmed residual evidence.**

---

## 8. PyInstaller

PyInstaller provides another important implementation-specific behavior.

Its code explicitly rejects non-CPython implementations.

At the same time, its packaging artifacts can use Python-generic tags for some distributions.

### Why this matters

PyInstaller is useful because it is not a trivial example of a pure-Python package with an accidental restriction.

It represents tooling whose behavior can depend substantially on the interpreter implementation.

This makes it a potentially valuable high-impact case for determining whether implementation support metadata could help consumers avoid unsuitable candidates.

### Root-cause interpretation

The current classification is not yet sufficiently narrow to treat the case as a final residual.

Potential dimensions include:

```text
A — runtime implementation dependence
B — build/toolchain behavior
D — ABI/configuration
E — implementation-specific APIs
F — explicit support policy
```

The case therefore requires release-specific auditing.

### Evidence status

**High-value candidate requiring deeper root-cause and release-level validation.**

---

## 9. PageBloomFilter build-backend case

PageBloomFilter provides an example where a native extension is described as currently supporting CPython only.

### Why this matters

This is useful because it separates:

```text
native extension support
```

from:

```text
generic pure-Python package behavior
```

The relevant question is whether the implementation boundary should instead be expressed through:

* wheel tags;
* build requirements;
* ABI metadata;
* build-system metadata;
* or another artifact-level mechanism.

### Root-cause interpretation

The current evidence places the case near:

```text
B — build/toolchain restriction
D — ABI/configuration restriction
```

rather than automatically classifying it as release-level support metadata.

### Evidence status

**Adjacent/boundary case.**

---

# 10. Packages that adapt to implementations

Not every implementation difference is a hard compatibility boundary.

Projects such as:

* `aiohttp`;
* `multidict`;
* `coverage.py`;
* `python-zstandard`;

provide examples of software that can adapt to different environments, provide fallbacks, or conditionally use implementation-specific features.

### Why this is important counter-evidence

A package can contain implementation-specific code without declaring an implementation-level exclusion.

For example:

```text
implementation-specific feature
        ↓
fallback available
        ↓
multiple implementations supported
```

Therefore:

```text
implementation-specific source code
        ≠
implementation unsupported
```

and:

```text
implementation-specific dependency
        ≠
top-level package unsupported
```

### Historical support transitions

`coverage.py` is especially useful as a temporal counterexample because its support for PyPy has changed across releases.

Its release history records changes such as adding and later confirming support for newer PyPy versions, while later releases no longer necessarily require a PyPy-specific wheel.

This demonstrates that implementation support can change over time without implying that a package needs a permanent implementation-specific artifact.

### Research implication

Any proposed metadata field must allow support relationships to vary by release.

It must also avoid forcing projects with conditional or fallback behavior into a false binary model such as:

```text
CPython = yes
PyPy = no
```

where the actual support relationship is more nuanced.

### Evidence status

**Counter-evidence / temporal design constraint.**

---

# 11. Historical implementation-support transitions

Historical releases demonstrate that implementation support is not necessarily static.

Requests, for example, has explicitly documented changes in its supported PyPy versions across releases.

NumPy also provides historical evidence of PyPy support evolving alongside compatibility work in its C-API ecosystem.

These examples establish:

```text
release R1:
    implementation support state A

release R2:
    implementation support state B
```

### Why this matters

This supports the idea that any implementation-support information, if standardized, would naturally need **release-level semantics**.

It also creates a maintenance question:

> Can producers reliably update implementation support declarations whenever support changes?

Historical transitions therefore provide evidence for both:

```text
need for precise release semantics
```

and:

```text
risk of stale declarations
```

### Evidence status

**Historical evidence / semantic and maintenance constraint.**

---

# 12. Trove classifiers

Projects can already use classifiers such as:

```text
Programming Language :: Python :: Implementation :: CPython
```

and:

```text
Programming Language :: Python :: Implementation :: PyPy
```

The classifier vocabulary also includes other implementations such as GraalPy, IronPython, Jython, MicroPython, and Stackless.

### Why this matters

The ecosystem already has a vocabulary for describing implementation targeting.

Therefore the research question is not:

```text
Can implementation identity be represented?
```

It can.

The relevant question is:

```text
Can implementation identity be represented with
precise machine-actionable compatibility semantics?
```

### Controlled experiment

A synthetic package containing:

```text
CPython implementation classifier
+
Requires-Python >=3.8
+
py3-none-any wheel
```

was tested with pip using an explicitly simulated PyPy target.

The pip path still selected the generic wheel.

A corresponding uv offline resolution test also resolved the generic artifact, although the tested uv interface did not provide an explicit PyPy implementation override. That result therefore must not be interpreted as a complete PyPy-specific uv experiment.

The experiment supports the narrower conclusion:

> Existing implementation classifiers are not presently equivalent to normative implementation compatibility constraints in the tested packaging paths.

### Important qualification

This does not prove that classifiers are inadequate for all use cases.

They may remain sufficient for:

* human discovery;
* project classification;
* package-index browsing;
* ecosystem analysis;
* test-matrix hints.

The unresolved question is whether a separate normative representation provides enough additional value to justify its cost.

### Evidence status

**Existing mechanism + important semantic counterpoint.**

---

# 13. Sdist build avoidance

Packaging discussions have identified cases where an installer may select an sdist and attempt a build that is likely to fail or is not intended for ordinary installation.

The general problem is:

```text
source distribution
        ↓
unknown build characteristics
        ↓
expensive build attempt
        ↓
failure or unsuitable result
```

### Relationship to implementation support

An implementation-specific build failure may arise from:

```text
implementation bug
build dependency
host dependency
compiler/toolchain
ABI
private implementation usage
runtime support policy
```

These should not be collapsed into a single metadata concept.

PEP 725 is relevant to build and host requirements.

Therefore:

```text
build failure
    ≠
implementation support residual
```

without further root-cause evidence.

### Evidence status

**Relevant adjacent problem.**

It establishes the practical value of avoiding unnecessary source builds, but does not establish that implementation support is the correct metadata layer.

---

# 14. Pure-Python detection discussion

A separate packaging discussion asked how tooling could determine whether an sdist is pure Python without attempting a complete build.

This provides another example of a broader packaging problem:

```text
source distribution
        ↓
unknown build characteristics
        ↓
tool must perform work to discover them
```

### Why it matters

This problem helps distinguish:

```text
implementation compatibility
        +
build characteristics
        +
artifact compatibility
```

They may interact, but they are not the same property.

For example:

```text
pure Python
```

does not necessarily mean:

```text
all Python implementations supported
```

Likewise:

```text
implementation-specific build
```

does not necessarily mean:

```text
implementation unsupported
```

### Research implication

A proposed implementation-support field should not become a general-purpose replacement for build metadata.

Each piece of information should remain in the mechanism whose semantics match the information being represented.

### Evidence status

**Adjacent evidence / boundary condition.**

---

# 15. Support inheritance and dependency graphs

Some implementation restrictions arise below the top-level package.

For example:

```text
package A
    |
    +--> dependency B
            |
            +--> CPython-specific component
```

It is unsafe to infer:

```text
B supports CPython only
        ↓
A supports CPython only
```

because A may:

* conditionally select B;
* provide an alternative dependency;
* provide a fallback implementation;
* use B only for an optional feature;
* or otherwise remain compatible with multiple implementations.

This is why wrapper-style projects and packages with conditional dependencies are useful control cases.

### Research implication

Implementation support is a property of the release's **effective behavior**, not necessarily a property that can be derived transitively from one dependency.

A future metadata mechanism should therefore represent an explicit producer declaration rather than invite consumers to infer top-level support transitively.

### Evidence status

**Control / design constraint.**

---

# 16. Failed-build caching as an alternative

Another relevant observation from packaging discussion is that tooling can cache failed build results.

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

This is relevant to the argument that metadata might be useful for avoiding expensive failed source builds.

### Why caching is not equivalent to support metadata

A cached failure records:

```text
observed result in environment X
```

whereas support metadata would represent:

```text
producer declaration about release R
```

A cache also has limitations:

* the first attempt still occurs;
* the result may be environment-specific;
* a failure may be caused by a temporary issue;
* a later release may fix the problem;
* the result may not transfer safely between environments.

Therefore caching is a credible alternative for the **performance** aspect of failed builds, but it does not answer the semantic question of whether a release supports an implementation.

### Evidence status

**Alternative / counterargument.**

This weakens the claim that a new field is automatically necessary merely because source builds can fail.

---

# 17. Implementation support can change between releases

The historical cases demonstrate an additional property that is important for any proposed field:

```text
implementation support
    is release-specific
```

A project can:

```text
release R1:
    CPython only

release R2:
    CPython + PyPy

release R3:
    CPython + PyPy + another implementation
```

or move in the opposite direction.

This is consistent with the general structure of Python package releases, where compatibility is already represented per release through:

* `Requires-Python`;
* wheel availability;
* artifact tags;
* metadata;
* project classifiers.

### Research implication

A support declaration should not be interpreted as a permanent project-level property.

If such a field were introduced, it would need to attach unambiguously to the release metadata being evaluated.

### Evidence status

**Important release-level design constraint.**

---

# 18. Implementation-specific support is not necessarily transitive

A top-level package can support an implementation even when one optional dependency does not.

Conversely, a top-level package can be incompatible even when all of its direct dependencies individually support the implementation.

Therefore:

```text
dependency support
        ≠
top-level release support
```

without understanding how the dependency is used.

This limits the usefulness of attempting to derive implementation support automatically from dependency metadata.

### Research implication

If implementation support becomes standardized, the declaration would most likely need to be **producer-authored**, rather than automatically calculated by a resolver from dependency declarations.

That would increase semantic value but also increase the producer-maintenance burden.

### Evidence status

**Control / design constraint.**

---

# 19. Research interpretation

The current real-world evidence supports several observations:

* implementation-specific compatibility is real;
* implementation-specific runtime logic is real;
* implementation-specific build logic is real;
* some releases explicitly communicate implementation restrictions;
* some releases enforce implementation restrictions at runtime;
* generic wheel artifacts can coexist with narrower producer support statements;
* a pure-Python or Python-generic wheel does not necessarily imply universal producer support;
* source distributions expose a release-level compatibility question that wheel tags do not always answer before a build;
* implementation information already exists descriptively through classifiers;
* tested resolver behavior does not currently treat implementation classifiers as generic hard compatibility constraints;
* packages can adapt to multiple implementations through fallbacks;
* implementation restrictions can arise from dependencies, ABI, build systems, private APIs, security requirements, or alternative-interpreter limitations;
* implementation support can change between releases;
* failed-build caching provides an alternative for some performance-oriented cases.

The evidence does **not yet establish**:

* that implementation restrictions are common enough to justify new metadata;
* that all explicit CPython-only declarations represent technical incompatibility;
* that every runtime guard should become a package-level compatibility constraint;
* that installers should reject unsupported implementations;
* that classifiers are insufficient for every meaningful consumer;
* that PEP 725 cannot address the relevant build cases;
* that wheel tags or variants cannot represent the important artifact cases;
* that a new Core Metadata field is preferable to index metadata, artifact metadata, or other mechanisms;
* that `Requires-Implementation` is the correct semantic model;
* that `Supported-Implementation` is necessary.

---

# 20. Current evidence hierarchy

The cases in this document should be interpreted according to the following rough hierarchy:

```text
Level 1 — normative specification
    ↓
Level 2 — published release metadata
    ↓
Level 3 — producer documentation
    ↓
Level 4 — source/runtime behavior
    ↓
Level 5 — community discussion
```

A producer statement is strong evidence that the producer intends a particular support policy.

A runtime guard is strong evidence that a restriction is operationally enforced.

A published artifact is evidence of what was actually distributed.

A classifier is evidence of descriptive metadata chosen by the producer.

A community discussion is evidence of proposed use cases or ecosystem concerns, but is not by itself evidence of package behavior.

No single layer alone proves that a restriction belongs in normative package metadata.

The final standardization question requires evidence from all relevant layers.

---

# 21. Current classification of the strongest cases

The current corpus can be summarized qualitatively as:

| Case                 | Primary evidence                             | Current root-cause direction         | Current status       |
| -------------------- | -------------------------------------------- | ------------------------------------ | -------------------- |
| HAX 0.3.0            | Explicit runtime CPython guard               | A — runtime restriction              | Strong candidate     |
| simple-ctx-log 0.0.3 | CPython-only statement + `sys._getframe`     | E/F — private API + support boundary | Strong candidate     |
| RestrictedPython 8.5 | Explicit CPython/security support boundary   | F — support/security policy          | Strong policy case   |
| Likepy 0.3.0         | Explicit CPython-only statement              | Unresolved                           | Candidate            |
| TribeCore 4.7.3      | Generic Python tags + explicit support range | F — support policy                   | Control case         |
| Guppy3               | Implementation and configuration checks      | D/E                                  | Boundary/control     |
| Specialist           | Explicit CPython/version restriction         | A/F pending validation               | Candidate            |
| PyInstaller          | Explicit non-CPython rejection               | A/B/D/E/F pending validation         | High-value candidate |
| PageBloomFilter      | CPython-only native-extension support        | B/D                                  | Adjacent case        |

This table is intentionally qualitative.

It does not assign numerical confidence or prevalence without a reproducible corpus and explicit methodology.

---

# 22. Current conclusion

The real-world corpus establishes a meaningful phenomenon:

```text
release-level producer support
        can be narrower than
artifact-level compatibility metadata
```

The most interesting instances are releases where:

```text
producer:
    implementation-specific support boundary

artifact:
    implementation-generic wheel

sdist:
    available

Requires-Python:
    insufficient to express implementation identity
```

The strongest current runtime example is HAX 0.3.0.

Other examples provide different forms of evidence:

```text
simple-ctx-log
    → private implementation API + explicit support boundary

RestrictedPython
    → explicit security/support boundary

Likepy
    → explicit policy but unresolved root cause

TribeCore
    → useful support-policy control case

Guppy3
    → ABI/configuration boundary case
```

These cases demonstrate that:

```text
implementation support
        ≠
Python-version compatibility
        ≠
artifact compatibility
        ≠
observed compatibility
        ≠
build compatibility
```

However, the research is not yet at the point where these observations should be converted into a PEP recommendation.

The current working position is:

> There appears to be a genuine semantic distinction between artifact-level compatibility and producer-declared implementation support. Real releases demonstrate this distinction, but further evidence is required to determine whether it is sufficiently general, stable, actionable, and useful to justify a new normative metadata field.

The next step is therefore not to collect more examples indiscriminately.

It is to **adversarially validate the strongest examples**:

```text
real-world case
        ↓
exact release identified
        ↓
root-cause classification
        ↓
existing mechanisms tested
        ↓
false-positive explanations eliminated
        ↓
actual consumer decision identified
        ↓
pre-install/build timing established
        ↓
incremental benefit of new metadata measured
        ↓
producer-declaration trust assessed
```

Only cases that survive this process should be promoted into `evidence/residual-cases.md` as strong residual evidence.
