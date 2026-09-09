# Residual Cases

This file contains the most useful real-world releases found so far.

A **residual case** is not merely a package that says “CPython only”.

For this research, a release becomes a **strong residual case** only when the evidence survives the following checks:

1. the producer explicitly declares an implementation restriction or support boundary;
2. the concrete release/version is identified;
3. the release contains an sdist or otherwise presents a release-level candidate;
4. the published wheel is implementation-generic (`py3-...`) or does not otherwise encode the restriction;
5. `Requires-Python` cannot express the implementation distinction;
6. PEP 508 dependency markers do not express the distribution's own implementation support;
7. the restriction is not simply an ABI/configuration feature covered by PEP 780;
8. the restriction is not safely inferable from the absence of a wheel;
9. the root cause of the restriction is understood;
10. the restriction is not merely an alternative-interpreter bug or temporary workaround;
11. the case is not simply private implementation usage that the producer intentionally leaves unsupported;
12. a machine-readable release-level declaration could change a meaningful **pre-install candidate-selection or compatibility decision**.

The distinction between:

```text
observed restriction

        ≠

root cause

        ≠

standardization requirement
```

is important throughout this file.

The cases below are therefore classified conservatively. They are evidence for the research question, not proof that a new Core Metadata field is necessary.

---

# Case classification

The current taxonomy used for these cases is:

| Class | Root cause                                  | Relevance                                          |
| ----- | ------------------------------------------- | -------------------------------------------------- |
| A     | Runtime semantic restriction                | Potentially strong residual evidence               |
| B     | Build/toolchain restriction                 | May overlap with PEP 725                           |
| C     | Alternative-implementation bug/workaround   | Usually not evidence for new package metadata      |
| D     | ABI/configuration restriction               | Usually belongs to ABI/artifact mechanisms         |
| E     | Private implementation usage                | Usually not a clean normative support case         |
| F     | Explicit support-policy declaration         | Useful evidence, but must show consumer value      |
| G     | Conditional/fallback implementation support | Requires graph/runtime analysis                    |
| H     | Dependency/component restriction            | Must not automatically be inherited by the release |

A release can have more than one classification.

The current research question is not:

> “Can we find packages that say CPython only?”

It is:

> “Are there release-level implementation support boundaries that existing metadata cannot represent, for which a machine-readable declaration would materially improve pre-install decisions?”

---

# Case A — HAX 0.3.0

## Published release facts

PyPI publishes:

```text
hax-0.3.0.tar.gz
hax-0.3.0-py3-none-any.whl
```

The project states that HAX supports CPython 3.7+.

The 0.3.0 source contains an explicit implementation check:

```python
if implementation.name != "cpython":
    raise RuntimeError("HAX only supports CPython!")

if version_info < (3, 7):
    raise RuntimeError("HAX only supports Python 3.7+!")
```

The source therefore does not merely document a preference. The package explicitly rejects non-CPython implementations at runtime.

Historical release source:

```text
https://github.com/brandtbucher/hax/blob/add83a96a13458d66c42a8f58860e8fc25520fe4/hax/_checks.py
```

PyPI:

```text
https://pypi.org/project/hax/0.3.0/
```

## Why this is especially useful

This case contains several independent signals:

```text
published release
+
sdist
+
pure-Python wheel
+
implementation-generic wheel tag
+
producer documentation
+
source-level runtime enforcement
```

The artifact therefore does not encode the same implementation restriction that the package enforces.

The wheel is:

```text
py3-none-any
```

rather than a CPython-specific tag.

This makes HAX substantially stronger evidence than a package that merely has a CPython Trove classifier.

## Existing mechanisms

`Requires-Python` can represent:

```text
>=3.7
```

but cannot represent:

```text
implementation == cpython
```

The wheel tag:

```text
py3-none-any
```

also does not encode the implementation restriction.

PEP 508 implementation markers such as `implementation_name` can express environment-dependent dependency conditions, but they are not a declaration that the distribution itself is unsupported on another implementation.

The CPython Trove classifier communicates the producer's position descriptively, but current packaging behavior does not treat that classifier as a normative candidate exclusion rule.

## Controlled resolver experiment

A minimal wheel was constructed with:

```text
Name: classifier-only-cpython-test
Version: 1.0.0
Requires-Python: >=3.8
Classifier: Programming Language :: Python :: Implementation :: CPython
Wheel: py3-none-any
```

pip was then asked to resolve the package for a PyPy target:

```text
python3 -m pip download \
    --no-deps \
    --no-index \
    --find-links file:///.../dist \
    --implementation pp \
    --python-version 3.11 \
    classifier-only-cpython-test==1.0.0
```

pip selected the `py3-none-any` wheel successfully.

This controlled experiment demonstrates an important point:

```text
CPython Trove classifier
        ≠
normative pip implementation exclusion
```

It does **not** prove that every resolver behaves identically, nor does it by itself prove that a new metadata field is required. It establishes that the existing classifier is not sufficient to make the restriction machine-actionable for pip candidate selection.

## Root-cause classification

**Current classification: A/F candidate.**

The source-level runtime guard demonstrates an actual implementation restriction.

However, the guard alone does not establish whether the deeper cause is:

* runtime semantic dependence on CPython;
* private CPython APIs;
* an implementation-specific feature;
* an alternative-interpreter limitation;
* intentional support policy;
* or some combination.

Therefore the case should be described as **runtime-enforced implementation restriction**, while the underlying root cause remains a separate research question.

## Resolver consequence

Consider:

```text
Environment:
    PyPy 3.x

Candidate:
    hax 0.3.0

Existing metadata:
    Requires-Python may match
    py3-none-any wheel matches
    CPython classifier communicates the restriction

Actual package behavior:
    runtime rejection of non-CPython
```

A machine-readable implementation-support declaration could theoretically allow a resolver or installer to reject this candidate before installation.

That is the most concrete pre-install benefit demonstrated by the current residual-case research.

The remaining question is whether this situation is sufficiently prevalent and general to justify standardization.

## Residual status

**Strongest current residual candidate.**

HAX currently provides the cleanest observed example of the following combination:

```text
release-level support boundary
+
generic wheel
+
sdist
+
Requires-Python insufficient
+
classifier insufficient for normative filtering
+
explicit runtime enforcement
```

It should nevertheless not be treated as proof that a new metadata field is necessary.

---

# Case B — RestrictedPython 8.5

## Published release facts

PyPI lists:

```text
Requires-Python:
    >=3.10,<3.16

Source:
    RestrictedPython-8.5.tar.gz

Wheel:
    RestrictedPython-8.5-py3-none-any.whl

Classifier:
    Programming Language :: Python :: Implementation :: CPython
```

The project documentation explicitly states that RestrictedPython is supported only on CPython and not on PyPy or other Python implementations.

PyPI:

```text
https://pypi.org/project/RestrictedPython/
```

The current source contains an implementation check and warning explaining that RestrictedPython is only supported on CPython because using it on other implementations may create security issues.

The source also contains historical discussion of a PyPy-specific AST issue, showing that the implementation boundary has had both technical and security dimensions.

## Artifact/support mismatch

The wheel is:

```text
py3-none-any
```

Therefore:

```text
artifact compatibility:
    generic Python 3

producer support policy:
    CPython only
```

The implementation restriction is not encoded by the wheel tag.

## Root-cause classification

**Current classification: F — explicit support-policy/security boundary, with technical causes also relevant.**

This case differs from HAX.

The strongest evidence is not simply that the package happens to fail on another implementation. The producer explicitly states that supporting other implementations would not provide the security guarantees required by RestrictedPython.

That makes this an important example of a legitimate **support policy** that has operational consequences.

However, this also exposes a potential problem with treating support metadata as a hard resolver constraint:

```text
producer support statement
        ↓
machine-readable restriction
        ↓
installer rejection
```

A support statement can become stale even when the implementation becomes compatible later.

The research therefore needs to distinguish:

```text
known technical incompatibility
```

from:

```text
current producer support policy
```

and determine whether both should have identical metadata semantics.

## Existing mechanisms

| Mechanism          | Representation                                                        |
| ------------------ | --------------------------------------------------------------------- |
| `Requires-Python`  | Python version range only                                             |
| Wheel tag          | Generic `py3-none-any`                                                |
| PEP 508 markers    | Dependency applicability, not package self-support                    |
| CPython classifier | Descriptive support information                                       |
| PEP 780            | Not established as the appropriate mechanism                          |
| PEP 725            | Relevant only if the underlying issue is an external/build dependency |

The classifier therefore communicates the producer's position, but does not provide the same normative candidate-selection behavior as a dedicated compatibility field would.

## Resolver consequence

For:

```text
Environment:
    PyPy 3.11

Candidate:
    RestrictedPython 8.5

Existing metadata:
    Requires-Python matches
    py3-none-any matches
    classifier says CPython
```

the resolver currently has no dedicated package-self-support field that means:

```text
this release is not supported on PyPy
```

A machine-readable declaration could potentially avoid installation.

But the research must still establish whether this is desirable for support-policy statements generally.

## Residual status

**Strong candidate, but not yet a clean technical residual case.**

RestrictedPython is valuable because it demonstrates that the semantic distinction can arise from a deliberate and consequential support policy rather than only from an accidental runtime failure.

It also demonstrates the stale-declaration risk that any new normative support field would introduce.

---

# Case C — simple-ctx-log 0.0.3

## Published release facts

PyPI describes the implementation limitation in terms of:

```text
Uses sys._getframe (CPython only)
```

The release publishes:

```text
simple_ctx_log-0.0.3.tar.gz
simple_ctx_log-0.0.3-py3-none-any.whl
```

Release date:

```text
February 1, 2026
```

PyPI:

```text
https://pypi.org/project/simple-ctx-log/
```

The exact 0.0.3 source uses:

```python
def _find_caller_frame(self) -> FrameType | None:
    try:
        frame = sys._getframe(2)
    except ValueError:
        return None
```

Release source:

```text
https://github.com/Hactys/simple-ctx-log/tree/e14d5966105da046090871e8f493e143678a631d
```

## Why it matters

This is another independent example of:

```text
pure Python
+
implementation-specific API
+
py3-none-any
+
sdist
```

It therefore reproduces the same artifact/implementation-support mismatch seen in HAX.

However, it is important not to overstate what the source proves.

The use of `sys._getframe` establishes a dependency on a private CPython API. It does not, by itself, establish that:

```text
PyPy cannot provide equivalent behavior
```

or:

```text
the package necessarily fails on every alternative implementation
```

or:

```text
the producer would reject all alternative implementations even if the API were available
```

## Root-cause classification

**Current classification: E/F candidate — private implementation API plus explicit support boundary.**

The source validation strengthens the case considerably compared with documentation alone.

However, this is exactly the kind of case that must not automatically be converted into a new metadata requirement.

Private implementation usage may instead be handled by:

* implementation-specific documentation;
* compatibility work in the alternative implementation;
* a future public API;
* artifact metadata where appropriate;
* or simply project policy.

The existence of a private CPython API dependency is therefore evidence of an implementation distinction, but not automatically evidence of a metadata gap.

## Existing mechanisms

The wheel remains:

```text
py3-none-any
```

and therefore does not encode the implementation-specific dependency.

`Requires-Python` cannot express:

```text
CPython provides sys._getframe
```

PEP 508 can condition dependencies on implementation, but does not directly declare that the package itself is unsupported on another implementation.

## Residual status

**Residual candidate — technically stronger than documentation-only cases, but not yet a strong residual case.**

The remaining validation questions are:

1. Does the relevant alternative implementation actually lack the required semantics?
2. Does the package fail or become unusable there?
3. Is the support boundary intentional?
4. Would a pre-install decision materially benefit users?
5. Is this better understood as private-API usage rather than a general implementation-support declaration?

Until those questions are resolved, the case should remain below HAX.

---

# Case D — Likepy 0.3.0

## Published release facts

PyPI lists:

```text
Requires-Python:
    >=3.6,<3.12

Source:
    likepy-0.3.0.tar.gz

Wheel:
    likepy-0.3.0-py3-none-any.whl

Classifier:
    Programming Language :: Python :: Implementation :: CPython
```

The project description states that Likepy only supports CPython and does not support PyPy or other Python implementations.

PyPI:

```text
https://pypi.org/project/likepy/0.3.0/
```

## Existing mechanisms

The Python version restriction is represented by:

```text
Requires-Python: >=3.6,<3.12
```

The implementation restriction is not represented by the wheel:

```text
py3-none-any
```

The CPython classifier communicates the producer's support position descriptively.

## Root-cause classification

**Current classification: F — explicit support-policy declaration.**

The available evidence establishes a CPython-only producer statement.

It does not yet establish whether the restriction is:

* runtime-semantic;
* build-related;
* caused by alternative-interpreter limitations;
* caused by private implementation usage;
* or simply project support policy.

The source repository was not reliably retrieved during the current investigation, so the root cause must not be inferred.

## Resolver consequence

Potentially:

```text
Environment:
    non-CPython implementation

Candidate:
    likepy 0.3.0

Existing metadata:
    version range matches
    generic wheel matches
    classifier communicates CPython

Potential improvement:
    machine-readable implementation information
```

But the actual user benefit has not been demonstrated.

## Residual status

**Residual candidate — insufficient root-cause evidence for strong classification.**

Likepy is still useful because it independently reproduces the pattern:

```text
CPython-only producer statement
+
generic Python wheel
+
sdist
+
Python version restriction represented separately
```

It should not currently be counted as independent proof of a new metadata requirement.

---

# Case E — TribeCore 4.7.3

## Published release facts

PyPI states that the project is CPython-only and does not support PyPy or other Python implementations.

It publishes an sdist and wheels such as:

```text
tribecore-4.7.3-py3-none-win_amd64.whl
tribecore-4.7.3-py3-none-manylinux2014_x86_64.whl
tribecore-4.7.3-py3-none-macosx_13_0_universal2.whl
```

Released:

```text
August 6, 2026
```

The project explains that it uses a Zig native library accessed through `ctypes`, while using `py3-none-{platform}` so the wheels can work across supported Python 3 versions on a given platform.

PyPI:

```text
https://pypi.org/project/tribecore/
```

## Why it matters

This demonstrates that the phenomenon is not limited to:

```text
py3-none-any
```

A wheel can be platform-specific while remaining implementation-generic:

```text
py3-none-win_amd64
py3-none-manylinux...
py3-none-macosx...
```

Therefore:

```text
platform compatibility
        ≠
implementation support
```

can remain separate dimensions.

## Root-cause classification

**Current classification: B/D/F/H candidate.**

The use of native/bundled components means the apparent CPython-only boundary could arise from:

* build/toolchain constraints;
* ABI/configuration;
* native component behavior;
* bundled dependencies;
* explicit support policy;
* or a combination.

The current evidence does not justify assigning one of these as the definitive root cause.

## Resolver consequence

The interesting environment is:

```text
Environment:
    PyPy + Linux x86_64

Candidate:
    tribecore 4.7.3

Wheel:
    py3-none-manylinux2014_x86_64

Producer:
    CPython only
```

If the wheel is technically usable by PyPy but the producer deliberately does not support PyPy, this could represent a genuine release-level support distinction.

But if the actual restriction is caused entirely by native/ABI/build mechanics, the case may belong to existing artifact or build mechanisms rather than a new implementation-support field.

## Residual status

**Residual candidate — important boundary case, but not yet clean evidence.**

The native/bundled component explanation must be resolved before this case is counted as strong residual evidence.

---

# Case F — winuvloop 0.2.5

## Published release facts

PyPI publishes:

```text
winuvloop-0.2.5.tar.gz
winuvloop-0.2.5-py3-none-any.whl
```

Released:

```text
August 24, 2026
```

## Why it is useful

This case is primarily a test of **support inheritance**.

A dependency graph may look like:

```text
package A
    |
    +--> implementation-specific package B
```

It is unsafe to automatically infer:

```text
B supports CPython only
        ↓
A supports CPython only
```

because A could:

* provide a fallback;
* select different dependencies conditionally;
* support multiple implementations through different paths;
* or impose a different compatibility boundary.

## Root-cause classification

**Current classification: H — dependency/component restriction.**

The wrapper itself should not be classified as CPython-only without analyzing its complete dependency graph and runtime behavior.

## Residual status

**Mixed / not currently a clean residual case.**

It is primarily a negative control against incorrectly propagating implementation restrictions through dependencies.

---

# Control cases

## psutil

psutil contains implementation-specific build logic but supports multiple Python implementations.

This demonstrates:

```text
implementation-specific source/build logic
        ≠
implementation unsupported
```

The research must therefore avoid classifying every occurrence of:

```python
sys.implementation
```

or implementation-specific build code as evidence of an unsupported implementation.

**Classification: control.**

---

## Autobahn

Autobahn supports multiple Python implementations while some optional/native components have narrower implementation support.

This demonstrates:

```text
CPython-only dependency/component
        ≠
CPython-only release
```

Implementation support cannot safely be inferred transitively.

**Classification: control.**

---

## Guppy3

Guppy3 is a strong implementation/ABI example, but its CPython-specific wheels already encode substantial artifact-level information.

It is therefore more useful for studying the boundary between:

```text
implementation identity
ABI configuration
Python version
artifact compatibility
```

than as a clean residual case.

**Classification: D / artifact-compatibility control.**

---

## bocpy

bocpy is a strong CPython/private-ABI example.

Its compatibility depends on CPython-specific private APIs and Python-version details.

It demonstrates why an implementation-support mechanism would not replace:

```text
Python version metadata
+
ABI metadata
+
artifact compatibility metadata
```

**Classification: D/E control or boundary case.**

---

# Additional runtime-guard candidates

The investigation has also identified additional packages with explicit implementation checks, including:

* `brandtbucher/specialist`, which contains an explicit CPython-only runtime restriction;
* PyInstaller, which contains explicit checks for CPython;
* PageBloomFilter's build/backend documentation, which states that its native extension currently supports CPython only.

These are **candidate discoveries**, not yet strong residual cases.

They should not be promoted merely because they contain:

```python
sys.implementation.name == "cpython"
```

The same residual-case gate must be applied:

```text
exact release
+
artifact metadata
+
root cause
+
existing mechanisms
+
consumer consequence
```

This is important because a larger list of examples is not useful if the examples all reduce to ABI, private APIs, build tooling, or alternative-interpreter bugs.

---

# Evidence-status summary

| Release              | Producer claim                              | Artifact mismatch                 | Current root cause                         | Resolver consequence demonstrated?                           | Status                    |
| -------------------- | ------------------------------------------- | --------------------------------- | ------------------------------------------ | ------------------------------------------------------------ | ------------------------- |
| RestrictedPython 8.5 | CPython only                                | `py3-none-any`                    | F/security; technical causes also possible | Potentially                                                  | **Strong candidate**      |
| HAX 0.3.0            | CPython only                                | `py3-none-any`                    | A/F candidate                              | Yes, conceptually and through controlled metadata comparison | **Strongest candidate**   |
| simple-ctx-log 0.0.3 | CPython only                                | `py3-none-any`                    | E/F candidate                              | Potentially                                                  | Candidate                 |
| Likepy 0.3.0         | CPython only                                | `py3-none-any`                    | F; root cause unresolved                   | Potentially                                                  | Candidate                 |
| TribeCore 4.7.3      | CPython only                                | `py3-none-{platform}`             | B/D/F/H possible                           | Potentially                                                  | Candidate / boundary case |
| winuvloop 0.2.5      | Indirect implementation-specific dependency | `py3-none-any`                    | H                                          | No                                                           | Mixed / control           |
| psutil               | Multi-implementation                        | Implementation-specific internals | Not a restriction                          | No                                                           | Control                   |
| Autobahn             | Multi-implementation                        | Component-level restrictions      | H                                          | No                                                           | Control                   |
| Guppy3               | CPython/ABI-specific                        | More strongly encoded in wheels   | D                                          | Not the target problem                                       | Control                   |
| bocpy                | CPython/private ABI                         | Version/ABI-sensitive             | D/E                                        | Not cleanly                                                  | Control                   |

The current evidence should therefore **not** be summarized as “five strong residual cases”.

A more accurate description is:

```text
1 strongest technical residual candidate
+
1 strong support-policy/security candidate
+
several unresolved or boundary candidates
+
multiple negative controls
```

This distinction is important for standards work.

---

# What the controlled classifier experiment establishes

The classifier experiment deserves separate treatment because it tests an existing proposed solution directly.

A wheel containing:

```text
Classifier:
    Programming Language :: Python :: Implementation :: CPython
```

and:

```text
Wheel:
    py3-none-any
```

was supplied to pip with a PyPy implementation target.

pip selected the wheel.

The result establishes:

```text
Trove implementation classifier
        ≠
hard candidate compatibility constraint in pip
```

This supports the proposition that classifiers are currently descriptive rather than normative installation metadata.

The experiment does **not** establish:

```text
therefore a new Core Metadata field is necessary
```

because alternative solutions remain possible, including:

* improved resolver interpretation of existing metadata;
* richer wheel/artifact tagging where the restriction is artifact-derived;
* improved build avoidance;
* failed-build caching;
* package-specific compatibility mechanisms;
* or simply accepting that support-policy declarations are informational.

The experiment therefore proves a **current tooling gap**, not yet a **standardization requirement**.

---

# What would promote a case to a strong residual case?

A case should be promoted only when the following chain is established:

```text
1. Producer explicitly states support boundary
                    ↓
2. Concrete release is identified
                    ↓
3. Release-level candidate exists
                    ↓
4. Existing artifact metadata does not encode the boundary
                    ↓
5. Requires-Python cannot encode it
                    ↓
6. PEP 508 does not encode self-support
                    ↓
7. PEP 780 / ABI mechanisms do not already solve it
                    ↓
8. Root cause is classified
                    ↓
9. Alternative explanations are eliminated
                    ↓
10. A consumer encounters a meaningful pre-install decision
                    ↓
11. New machine-readable metadata would improve that decision
                    ↓
12. The benefit is sufficiently general to justify standardization
```

Failure at steps 8–12 means the case remains evidence of a **semantic mismatch**, but not yet evidence that a new standard is necessary.

Step 12 is important.

Even if a package clearly has a support boundary that existing metadata cannot encode, that alone does not justify a new field. The research must establish that the same problem occurs often enough, with sufficiently stable semantics, and that consumers would actually use the information.

---

# Important non-inferences

The following conclusions must **not** be drawn from these cases.

## No wheel for an implementation

```text
No PyPy wheel
    ≠
PyPy unsupported
```

A project may publish wheels only for the most common implementation while still supporting source installation elsewhere.

---

## CPython classifier

```text
CPython classifier
    ≠
formal statement that all other implementations are incompatible
```

The classifier is currently treated as descriptive evidence.

The controlled pip experiment additionally demonstrates that the classifier is not sufficient by itself to prevent candidate selection.

---

## Runtime implementation check

```text
sys.implementation check
    ≠
proof that a new metadata field is required
```

The underlying reason for the check must still be understood.

A runtime check can reflect:

* actual semantic dependence;
* private API usage;
* ABI assumptions;
* an implementation bug;
* a temporary workaround;
* or project policy.

---

## Native extension

```text
native extension
    ≠
implementation-support metadata problem
```

The restriction may already belong to:

* wheel tags;
* ABI information;
* build requirements;
* external dependencies;
* or platform metadata.

---

## Implementation-specific dependency

```text
dependency supports CPython only
    ≠
top-level package supports CPython only
```

Support can be conditional or provide fallbacks.

---

## Failed build

```text
one failed build
    ≠
normative unsupported declaration
```

A failed build may result from:

* a temporary toolchain issue;
* an alternative-interpreter bug;
* a missing external dependency;
* an unavailable compiler;
* a configuration problem;
* or another environmental condition.

PEP 725 should therefore be considered whenever the apparent restriction is actually an external build/host/runtime dependency problem.

---

## Support policy

```text
producer says "unsupported"
    ≠
technically impossible
```

A producer may intentionally support only one implementation even when another implementation happens to run the package correctly.

If a future metadata field makes such declarations normative, the standard must decide whether it represents:

```text
technical incompatibility
```

or:

```text
producer support policy
```

or whether those concepts require different semantics.

---

# The stale-declaration problem

A new normative support field would introduce a failure mode that existing artifact tags generally avoid:

```text
package becomes compatible
        ↓
producer forgets to update support metadata
        ↓
resolver rejects otherwise usable candidate
```

The reverse can also occur:

```text
package becomes incompatible
        ↓
producer forgets to narrow support metadata
        ↓
resolver accepts candidate
        ↓
failure occurs later
```

This creates a central standards-design question:

> Is producer-declared implementation support sufficiently objective and maintainable to serve as normative resolver input?

The residual cases do not answer this yet.

RestrictedPython is particularly relevant because its boundary is explicitly a security/support-policy decision.

HAX is relevant from the opposite direction because the package enforces the boundary at runtime.

The two cases therefore should not automatically receive identical metadata semantics.

---

# Historical support transitions

Historical releases show that implementation support can change over time.

Examples include:

* Requests adding and later changing official PyPy support across releases;
* coverage.py adding support for particular PyPy versions and later changing its wheel strategy;
* NumPy adding PyPy support through a C-API compatibility layer.

These examples demonstrate:

```text
implementation support can be release-specific
```

which is relevant to release-level metadata.

However, they do not establish that a new field is necessary.

Historical support transitions can also be communicated through:

* documentation;
* classifiers;
* release notes;
* wheel publication;
* source/build behavior;
* and dependency metadata.

The remaining research question is whether those mechanisms are insufficient for an important consumer decision.

---

# Current residual-case conclusion

The research has established a recurring semantic pattern:

```text
producer:

    CPython only

artifact:

    py3-...

sdist:

    present

Requires-Python:

    expresses Python version, not implementation identity

PEP 508:

    expresses dependency applicability, not package self-support

classifier:

    can communicate the producer's position descriptively

normative release-level implementation support:

    not currently represented
```

This is a real phenomenon.

The controlled classifier experiment strengthens the tooling-gap portion of the argument:

```text
descriptive implementation classifier
        ≠
normative candidate exclusion
```

However, the cases do **not yet establish** that a new Core Metadata field is necessary.

The most important unresolved chain remains:

```text
observed support restriction
        ↓
root cause
        ↓
existing mechanism that should represent it
        ↓
actual consumer decision
        ↓
incremental value of new metadata
        ↓
prevalence/generalizability
        ↓
standardization requirement
```

The strongest current technical example is:

```text
HAX 0.3.0
```

because it combines:

```text
CPython-only producer statement
+
explicit runtime enforcement
+
py3-none-any wheel
+
sdist
+
Requires-Python insufficiency
+
implementation classifier insufficiency for pip candidate filtering
```

RestrictedPython 8.5 is the strongest current **support-policy/security** example.

simple-ctx-log 0.0.3 is a useful **private-API/implementation-boundary** example.

Likepy remains a useful **documentation/support-policy** candidate.

TribeCore remains an important **native/artifact boundary** case.

The other cases are primarily controls against overgeneralization.

The current evidence therefore supports the following research position:

> There appears to be a genuine distinction between artifact-level compatibility and producer-declared implementation support. Several releases expose that distinction in practice, and current implementation classifiers are not sufficient to make the distinction normative for pip candidate selection. The remaining question is whether the distinction is sufficiently well-defined, stable, prevalent, and useful to justify a new normative, machine-actionable metadata field.

The research should **not** yet recommend either:

```text
Supported-Implementation
```

or:

```text
Requires-Implementation
```

as the solution.

The next investigation should focus on:

1. validating the root cause of each remaining candidate;
2. determining whether each restriction is runtime, build, ABI, dependency, private-API, or support-policy driven;
3. testing alternative implementations where execution evidence is feasible;
4. determining whether PEP 725 or existing artifact mechanisms already solve the relevant cases;
5. demonstrating a concrete resolver/index decision that would improve with machine-readable implementation support metadata;
6. determining whether the same decision recurs across sufficiently many independent projects;
7. evaluating stale or overly restrictive producer declarations as a first-class failure mode;
8. determining whether support policy and technical incompatibility should have the same metadata semantics;
9. comparing a new field against non-metadata alternatives such as failed-build caching and improved resolver/build avoidance;
10. measuring actual ecosystem adoption of implementation classifiers before assuming that producers would populate a new field.

Until those questions are answered, the residual cases should be treated as **research evidence rather than a recommendation for `Supported-Implementation` or `Requires-Implementation`**.
