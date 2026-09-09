# Research Status — 2026-09-09

## Current conclusion

The research has now established a stronger version of the original problem statement:

> Python implementation identity is a real compatibility dimension that is not equivalent to Python version compatibility or wheel artifact compatibility.

Real releases can combine:

```text
implementation-specific producer support
+
implementation-generic Python wheel tags
+
a separate Requires-Python constraint
```

The strongest current example is HAX 0.3.0, where the published release includes a `py3-none-any` wheel while the package source explicitly rejects non-CPython implementations at runtime.

However, the research has **not** established that this semantic gap requires a new Core Metadata field.

The central research principle remains:

```text
observed restriction
        ≠
root cause
        ≠
metadata gap
        ≠
need for a new standard
```

A package saying “CPython only” is therefore not automatically evidence that a new metadata field is necessary.

---

# The refined research question

The question is no longer:

> Do we need `Requires-Implementation`?

The working question is:

> **Are there real release-level Python implementation support constraints for which existing classifiers, artifact tags, build metadata, dependency metadata, and cached build outcomes are insufficient, and where standardized machine-readable metadata would materially improve a consumer's pre-install or candidate-selection decision?**

A second question follows:

> **If such cases exist, can existing implementation classifiers safely provide the required semantics, or is a new standardized metadata mechanism justified?**

The research must remain willing to conclude:

```text
no new standard
```

if the evidence does not cross the decision threshold.

---

# What is now established

## 1. Implementation identity is distinct from Python version

`Requires-Python` can express constraints such as:

```text
>=3.10,<3.14
```

but does not directly express:

```text
implementation == cpython
```

Therefore a release may have:

```text
Requires-Python: >=3.10
```

while still having an implementation-specific support boundary.

This distinction is real and not merely theoretical.

---

## 2. Artifact compatibility and producer support are distinct

Wheel tags describe the compatibility of a particular artifact.

A wheel such as:

```text
py3-none-any
```

is implementation-generic.

That does **not** logically prove:

```text
all Python implementations are supported
```

Likewise:

```text
no PyPy wheel
```

does not prove:

```text
PyPy unsupported
```

A producer may support source installation without publishing a dedicated wheel.

Therefore:

```text
artifact compatibility
```

and:

```text
producer-declared release support
```

must not be conflated.

---

# 3. Runtime-enforced implementation restrictions exist

The HAX 0.3.0 source contains an explicit implementation check:

```python
if implementation.name != "cpython":
    raise RuntimeError("HAX only supports CPython!")
```

The same release publishes:

```text
hax-0.3.0-py3-none-any.whl
```

and an sdist.

This is particularly valuable evidence because the implementation restriction is not merely:

```text
documentation
```

or:

```text
Trove classifier
```

but is enforced by executable package code.

### Current HAX classification

```text
A — Runtime semantic restriction
and/or
F — Explicit support-policy declaration
```

The runtime guard is confirmed.

The deeper reason for the CPython restriction still needs to be separated into:

* actual CPython semantic dependence;
* private API usage;
* alternative-interpreter limitation;
* support policy;
* or another technical reason.

### Current status

**Strongest current residual candidate.**

It is not yet proof that a new metadata field is necessary.

---

# 4. simple-ctx-log provides an independent implementation-specific case

The simple-ctx-log 0.0.3 source uses:

```python
sys._getframe(...)
```

The release is published with:

```text
simple-ctx-log-0.0.3-py3-none-any.whl
simple-ctx-log-0.0.3.tar.gz
```

The project itself describes the `sys._getframe` dependency as CPython-specific.

This gives another pattern of:

```text
pure Python
+
implementation-specific API
+
generic Python wheel
+
source distribution
```

The source-level dependency is now established.

The remaining research question is whether this should be classified as:

```text
A — runtime semantic restriction
```

or:

```text
E — private implementation usage
```

or another category.

That distinction matters because private implementation usage may not be an appropriate reason for introducing a new normative metadata field.

### Current status

**Strong residual candidate pending final root-cause and consumer-benefit analysis.**

---

# 5. RestrictedPython demonstrates a different class of problem

RestrictedPython's current source contains:

```text
IS_CPYTHON
```

and emits a warning that using RestrictedPython on other Python implementations may create security issues.

The project explicitly states that RestrictedPython is only supported on CPython.

This is important because the implementation boundary is tied to the project's **security guarantee**, not merely convenience.

The relevant distinction is:

```text
"does not run"
```

versus:

```text
"cannot make the same security guarantee"
```

The latter is especially important for support metadata because a producer may reasonably declare a supported implementation set without claiming that every unlisted implementation is technically incapable of running the code.

### Current classification

```text
F — explicit support-policy declaration
```

with possible:

```text
A — runtime/security semantic restriction
```

still requiring deeper investigation.

### Current status

**Important candidate, but not yet a clean proof of a metadata requirement.**

---

# 6. Likepy remains unresolved

Likepy 0.3.0 declares CPython-only support and publishes:

```text
likepy-0.3.0-py3-none-any.whl
likepy-0.3.0.tar.gz
```

Its version restriction is separately expressed through `Requires-Python`.

However, the research does not yet have sufficiently strong source evidence establishing the underlying technical reason for its CPython-only declaration.

Therefore the repository deliberately does **not** count Likepy as a confirmed technical residual case.

### Current status

```text
F — explicit support-policy evidence
```

but:

```text
root cause unresolved
```

This is exactly the type of case that should prevent inflated evidence counts.

---

# 7. TribeCore is not automatically a CPython-only technical case

TribeCore 4.7.3 publishes platform-specific wheels such as:

```text
py3-none-win_amd64
py3-none-manylinux...
py3-none-macosx...
```

The project uses a native Zig library through `ctypes` and describes Python-version compatibility separately.

The presence of native code therefore does not automatically establish:

```text
CPython ABI restriction
```

The correct analysis is:

```text
native component
        ↓
artifact/platform boundary
        ↓
ABI requirements
        ↓
build requirements
        ↓
actual implementation restriction?
```

Until the final technical boundary is established, TribeCore should not be counted as proof of a missing implementation-support metadata field.

### Current status

**Artifact/support mismatch candidate, but root cause not strong enough for residual counting.**

---

# 8. Classifiers are not currently installer-enforced implementation constraints

The research distinguishes carefully between:

```text
pip reads/parses classifier metadata
```

and:

```text
pip uses implementation classifiers as normative candidate constraints
```

These are not the same claim.

The current investigation found no evidence that pip treats:

```text
Programming Language :: Python :: Implementation :: CPython
```

as equivalent to:

```text
Requires-Implementation: cpython
```

The same conclusion currently applies to the investigated uv behavior.

A controlled synthetic-package experiment further supports this distinction.

The test package had:

```text
Requires-Python: >=3.8
Classifier:
    Programming Language :: Python :: Implementation :: CPython
Wheel:
    py3-none-any
```

When pip was instructed to resolve for a PyPy implementation, the generic wheel remained a selectable candidate.

This demonstrates an important fact:

> The existing CPython Trove classifier is not currently an installer-enforced implementation compatibility constraint.

This does **not** imply that classifiers are useless.

They may still be useful as:

```text
descriptive producer information
```

or potentially as a future source of positive support semantics.

The unresolved question is whether changing their meaning would be safe.

---

# 9. Positive support semantics remain safer than negative inference

The current research continues to favor the following semantic hypothesis if new metadata is eventually justified:

```text
Supported-Implementation: cpython
```

rather than:

```text
Requires-Implementation: cpython
```

The distinction is intentional.

A positive declaration can mean:

```text
the producer explicitly declares support for CPython
```

without automatically implying:

```text
every implementation not listed is prohibited
```

The safest possible omission semantics remain:

```text
field absent
    =
no normative implementation-support declaration
```

rather than:

```text
field absent
    =
all implementations supported
```

or:

```text
field absent
    =
all implementations unsupported
```

This remains a **hypothesis**, not a recommendation.

---

# 10. Historical evidence shows implementation support can change by release

The research has found real examples where Python implementation support changes over project history.

Examples include projects that:

```text
initially lacked PyPy support
        ↓
added PyPy support
        ↓
changed their testing/build strategy
```

This matters because it demonstrates that implementation support is potentially a:

```text
release-level property
```

rather than a permanently project-wide property.

That strengthens the conceptual case for release metadata.

However, it does not by itself establish the need for a new Core Metadata field.

A release-level property can still be communicated through:

* documentation;
* classifiers;
* CI;
* artifact availability;
* project-specific metadata;
* index metadata;
* or other mechanisms.

The remaining question is whether a standardized machine-readable mechanism provides enough additional value.

---

# 11. PyPy and GraalPy research has not found ecosystem endorsement

The investigation looked for evidence that alternative Python implementation projects themselves are requesting or endorsing a new machine-readable package-support metadata field.

The current result is:

```text
no endorsement found
```

for both PyPy and GraalPy.

This must **not** be interpreted as:

```text
PyPy/GraalPy oppose the proposal
```

No such conclusion is supported.

The correct statement is:

> The research has not yet found evidence that the alternative implementation communities consider a new Core Metadata support field a necessary packaging mechanism.

This is relevant because implementation-support metadata would affect the ecosystem beyond CPython.

---

# 12. PEP 725 remains an important competing explanation

Some apparent implementation restrictions may actually be:

```text
build requirement
host requirement
external dependency
toolchain limitation
```

PEP 725 is therefore a first-class alternative.

The research must not convert:

```text
cannot build on implementation X
```

into:

```text
package requires implementation X
```

without determining the cause.

The correct decomposition is:

```text
source build
    ↓
build dependencies
    ↓
host dependencies
    ↓
external libraries/tools
    ↓
implementation-specific limitation
```

If the real problem is an external build/host dependency, PEP 725 may be the more appropriate metadata layer.

---

# 13. ABI/environment features are another competing mechanism

PEP 780 addresses environment ABI features such as:

```text
free-threading
debug builds
bitness
```

These are not equivalent to producer support policy.

Therefore a package that requires:

```text
some ABI/environment feature
```

should not automatically become evidence for:

```text
Supported-Implementation
```

The research continues to separate:

```text
implementation identity
```

from:

```text
ABI/environment feature
```

---

# 14. Wheel variants are another competing mechanism

PEP 825 addresses artifact variants.

This is relevant when the actual problem is:

```text
different artifacts for different environments
```

rather than:

```text
release-level producer support
```

The research therefore treats PEP 825 as an alternative explanation whenever a package's apparent implementation restriction can be represented through artifact selection.

---

# 15. Failed-build caching is a genuine competing solution

A resolver/build frontend could potentially remember:

```text
package X
version Y
implementation Z
build failed
```

and avoid repeating the same expensive build.

This could solve a substantial subset of the practical problem.

However, caching and support metadata are not semantically equivalent.

### Caching says:

```text
this attempt failed before
```

### Producer metadata says:

```text
the producer declares this implementation support boundary
```

A cache cannot necessarily help with a package such as HAX when:

```text
generic wheel installs
        ↓
runtime implementation check fails
```

because there is no source-build failure to cache.

Therefore:

```text
caching solves some cases
```

but:

```text
caching solves the entire problem
```

has been rejected.

The quantitative consumer-benefit comparison remains incomplete.

---

# Current root-cause taxonomy

Every candidate must be assigned to one or more categories:

| Class | Root cause                                  | Treatment                                                |
| ----- | ------------------------------------------- | -------------------------------------------------------- |
| A     | Runtime semantic restriction                | Potential residual                                       |
| B     | Build/toolchain restriction                 | Test against PEP 725                                     |
| C     | Alternative-implementation bug/workaround   | Usually not new metadata evidence                        |
| D     | ABI/configuration restriction               | Test against ABI/artifact mechanisms                     |
| E     | Private implementation usage                | Usually not a clean normative support case               |
| F     | Explicit support-policy declaration         | Potential evidence, but consumer value required          |
| G     | Conditional/fallback implementation support | Requires dependency/runtime analysis                     |
| H     | Dependency/component restriction            | Must not automatically become top-level support metadata |

The research must not count all categories equally.

---

# Current evidence ranking

| Question                                                      | Status                     | Confidence |
| ------------------------------------------------------------- | -------------------------- | ---------- |
| Implementation identity is distinct from Python version       | Established                | High       |
| Artifact compatibility differs from producer support          | Established                | High       |
| Generic wheels can coexist with CPython-only runtime behavior | Established                | High       |
| HAX contains an explicit CPython runtime guard                | Established                | High       |
| simple-ctx-log uses CPython-specific `sys._getframe`          | Established                | High       |
| RestrictedPython has a CPython/security support boundary      | Established                | High       |
| Likepy's underlying technical cause                           | Unresolved                 | Low/medium |
| TribeCore's actual implementation boundary                    | Unresolved                 | Medium     |
| Classifiers are installer-enforced constraints today          | No evidence found          | High       |
| Positive classifier semantics would be safe                   | Unresolved                 | Medium     |
| Failed-build caching solves some cases                        | Established conceptually   | High       |
| Failed-build caching solves all cases                         | Rejected                   | High       |
| PEP 725 covers some apparent cases                            | Established as alternative | High       |
| PyPI-wide prevalence                                          | Not measured               | Low        |
| Alternative-runtime ecosystem endorsement                     | Not found                  | Low/medium |
| New Core Metadata field is necessary                          | Not established            | Low/medium |

---

# Strong residual-case gate

A release should only be promoted to a strong residual case when all of the
following are substantially established:

```text
1. Exact release identified
        ↓
2. Producer support boundary established
        ↓
3. Relevant artifact/sdist identified
        ↓
4. Existing wheel tags do not encode the restriction
        ↓
5. Requires-Python cannot encode it
        ↓
6. PEP 508 cannot express the package's own support boundary
        ↓
7. ABI/environment mechanisms do not already solve it
        ↓
8. PEP 725 does not explain the actual cause
        ↓
9. Root cause is understood
        ↓
10. Alternative-interpreter bug/private-API explanations considered
        ↓
11. A real consumer encounters a meaningful pre-install decision
        ↓
12. New machine-readable metadata changes that decision
        ↓
13. The benefit is material enough to justify standardization
```

Failure at steps 8–13 means the case remains useful research evidence but should
not be presented as proof that a new standard is required.

---

# Current strongest candidates

## HAX 0.3.0

Current strength:

```text
█████████░
```

Why:

* explicit runtime guard;
* generic wheel;
* sdist;
* implementation distinction cannot be represented by `Requires-Python`;
* runtime failure is deterministic.

Remaining:

* underlying reason;
* direct alternate-runtime execution;
* consumer prevalence;
* whether existing metadata alternatives are sufficient.

---

## simple-ctx-log 0.0.3

Current strength:

```text
███████░░░
```

Why:

* concrete CPython-specific API;
* generic wheel;
* sdist;
* recent release.

Remaining:

* direct alternate-runtime behavior;
* classification as private API versus broader semantic restriction;
* concrete consumer-selection benefit.

---

## RestrictedPython 8.5

Current strength:

```text
███████░░░
```

Why:

* explicit CPython-only support;
* security-oriented support boundary;
* generic wheel.

Remaining:

* precise technical/security root cause;
* whether support policy should be normative resolver input;
* consumer-selection benefit.

---

## Likepy 0.3.0

Current strength:

```text
████░░░░░░
```

Why:

* explicit CPython-only declaration;
* generic wheel;
* sdist.

Remaining:

* underlying technical cause;
* actual consumer benefit.

---

## TribeCore 4.7.3

Current strength:

```text
████░░░░░░
```

Why:

* interesting generic implementation wheel tags;
* native component;
* explicit implementation support statement.

Remaining:

* determine whether the real boundary belongs to ABI, build, platform,
  dependency, or implementation support.

---

# What the research currently rejects

The evidence does **not** support any of these claims:

```text
"Every CPython-only package needs a new metadata field."
```

```text
"CPython classifiers should automatically become hard resolver constraints."
```

```text
"A generic wheel means all Python implementations are supported."
```

```text
"No PyPy wheel means PyPy is unsupported."
```

```text
"Native code means a package needs implementation metadata."
```

```text
"Any sys.implementation check proves a metadata gap."
```

```text
"Build failures prove implementation incompatibility."
```

```text
"Failed-build caching makes support metadata unnecessary."
```

```text
"PyPI contains a large percentage of implementation-restricted packages."
```

None of those claims has been established.

---

# The critical unresolved question

The strongest remaining question is now:

> **Can we demonstrate a sufficiently common, technically well-defined class of release-level implementation support boundaries where an installer or resolver would make a materially better decision before installation/build if standardized machine-readable support metadata existed?**

This is the decision point.

The existence of the semantic distinction is no longer the main uncertainty.

The uncertainty is:

```text
Does the distinction justify standardization?
```

---

# Required final research before PEP drafting

The next work should concentrate on the following, in this order.

## 1. Direct alternate-runtime execution

Run:

```text
HAX 0.3.0
simple-ctx-log 0.0.3
```

under:

```text
PyPy
GraalPy
```

where available.

Record:

```text
install result
import result
first failing operation
exception
whether the failure occurs before or after useful package behavior
```

Do not infer runtime behavior from documentation alone when execution is possible.

---

## 2. Finish root-cause audits

Complete:

```text
RestrictedPython
Likepy
TribeCore
```

using:

```text
source
CI
release metadata
issue tracker
documentation
```

and assign A–H classifications.

---

## 3. Measure classifier prevalence

Obtain a reproducible PyPI release/project-level dataset.

Report separately:

```text
projects with CPython classifier
releases with CPython classifier
projects with PyPy classifier
releases with PyPy classifier
generic-wheel correlation
classifier/documentation disagreement
```

Do not report enriched research-corpus percentages as PyPI prevalence.

---

## 4. Search historical support transitions

Find real projects where:

```text
CPython-only
        ↓
PyPy-supported
```

or:

```text
PyPy-supported
        ↓
support removed
```

and determine how the change was communicated.

This tests whether support declarations are sufficiently stable to become
normative metadata.

---

## 5. Search alternative-runtime issue trackers

Specifically look for:

```text
machine-readable implementation support
implementation metadata
PyPy package support metadata
GraalPy package support metadata
installer implementation constraints
```

The goal is to discover actual ecosystem demand rather than infer it.

---

## 6. Compare alternative solutions experimentally

For strong residual cases compare:

```text
A. current metadata
B. classifier interpretation
C. hypothetical support metadata
D. failed-build caching
E. PEP 725
F. artifact/variant mechanisms
G. index-level metadata
H. no new standard
```

Record the actual candidate-selection difference.

---

# Decision gate

A new standard should only be recommended if the evidence demonstrates:

```text
real residual cases
        +
appropriate root cause
        +
existing mechanisms insufficient
        +
meaningful pre-install decision
        +
material consumer benefit
        +
credible publisher adoption
        +
stable and understandable semantics
```

If any of those remain unsupported, the correct research conclusion may be:

```text
the semantic distinction exists,
but a new Core Metadata field is not justified.
```

That is a valid outcome.

---

# Accountability rule

Every major conclusion in this repository should remain auditable.

For each claim distinguish:

```text
OBSERVED
```

from:

```text
INFERRED
```

from:

```text
UNKNOWN
```

and record:

```text
alternative explanation tested
```

The purpose of this research is not to prove that
`Requires-Implementation` or `Supported-Implementation` is correct.

The purpose is to determine whether Python packaging actually needs a new
normative semantic layer for producer-declared implementation support.

At the current evidence level:

> **The semantic gap is real. The necessity of a new standard remains unproven.**
