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

# Case A — RestrictedPython 8.5

## Published release facts

PyPI lists:

```text
Requires-Python:

    >=3.10,<3.16

Source:

    restrictedpython-8.5.tar.gz

Wheel:

    restrictedpython-8.5-py3-none-any.whl

Classifier:

    Programming Language :: Python :: Implementation :: CPython
```

Source:

PyPI project page for RestrictedPython 8.5.

The project description explicitly states that RestrictedPython supports only CPython and not PyPy or other Python implementations.

## Artifact/support mismatch

The published wheel is:

```text
py3-none-any
```

The wheel's implementation tag is therefore generic rather than CPython-specific.

The release consequently presents two different signals:

```text
Artifact compatibility:

    generic Python 3 wheel

Producer support statement:

    CPython only
```

This is useful evidence because the implementation restriction is not encoded in the wheel filename.

## Existing mechanisms

| Mechanism            | Can it express the producer's restriction?                                                               |
| -------------------- | -------------------------------------------------------------------------------------------------------- |
| `Requires-Python`    | No — it expresses Python version requirements                                                            |
| PEP 508 markers      | No — they condition dependencies rather than directly rejecting the distribution itself                  |
| Wheel tag            | Not in this release — `py3-none-any` is implementation-generic                                           |
| Trove classifier     | Yes, descriptively                                                                                       |
| PEP 780              | Not obviously — this is not established as an ABI-feature restriction                                    |
| PEP 825              | Not at release level; it concerns wheel variants                                                         |
| PEP 725              | Potentially relevant to build-related causes, but does not by itself establish runtime support semantics |
| Failed-build caching | Could avoid repeated build attempts after failure, but does not declare support before the first attempt |

## Root-cause classification

**Current classification: F — explicit support-policy declaration, with A still requiring validation.**

The producer explicitly states CPython-only support.

However, the current evidence does not by itself establish *why* the package is CPython-only.

Possible explanations include:

* runtime semantic dependence on CPython;
* use of CPython-specific APIs;
* build-related limitations;
* alternative-interpreter compatibility problems;
* intentional project support policy;
* some combination of these.

The statement “CPython only” therefore cannot automatically be treated as proof of a normative runtime incompatibility.

## Resolver consequence

A potentially useful resolver question would be:

```text
Environment:

    PyPy 3.x

Candidate:

    RestrictedPython 8.5

Existing metadata:

    Requires-Python may match
    py3-none-any wheel may match
    CPython classifier may communicate the restriction

Question:

    Should the installer reject or deprioritize the candidate
    before installation/build?
```

The important unresolved question is whether that decision is sufficiently valuable and well-defined to justify machine-actionable metadata.

## Residual status

**Candidate residual case — root cause and consumer-selection benefit still require validation.**

It should no longer be labeled a fully established strong residual case solely from the published support statement.

---

# Case B — HAX 0.3.0

## Published release facts

PyPI lists:

```text
hax-0.3.0.tar.gz
hax-0.3.0-py3-none-any.whl
```

The project states that HAX supports CPython 3.7+ on all platforms.

The source contains an explicit runtime implementation check:

```python
if implementation.name != "cpython":
    raise RuntimeError("HAX only supports CPython!")

if version_info < (3, 7):
    raise RuntimeError("HAX only supports Python 3.7+!")
```

## Why this is especially useful

This case contains more than a classifier or documentation statement:

```text
published artifact
+
producer documentation
+
source-level runtime enforcement
```

The implementation distinction is therefore operationally significant.

The release also publishes:

```text
py3-none-any
```

so the wheel itself does not encode the CPython restriction.

## Existing mechanisms

`Requires-Python` can represent:

```text
>=3.7
```

but not:

```text
implementation == cpython
```

The wheel tag:

```text
py3-none-any
```

also does not encode CPython-only support.

PEP 508 dependency markers are not a direct representation of the package's own support boundary.

## Root-cause classification

**Current classification: A/F candidate.**

The runtime guard demonstrates an actual runtime restriction:

```text
if implementation.name != "cpython":
    raise RuntimeError(...)
```

This makes it stronger than a classifier-only example.

However, the research still needs to distinguish:

```text
runtime semantic restriction
```

from:

```text
private implementation usage
support policy
alternative-interpreter limitation
```

The guard proves enforcement, but not necessarily the underlying reason for the restriction.

## Resolver consequence

This case provides a clearer pre-install scenario:

```text
Environment:

    PyPy

Candidate:

    hax 0.3.0

Existing metadata:

    Python version may match
    py3-none-any wheel may match

Actual result:

    runtime failure due to implementation.name != "cpython"
```

A release-level implementation declaration could theoretically allow a consumer to avoid this candidate before installation.

Whether that benefit is sufficiently general and important remains an open research question.

## Residual status

**Strongest current residual candidate.**

It is particularly useful because:

* the restriction is explicitly enforced;
* the artifact is pure Python;
* the wheel is implementation-generic;
* `Requires-Python` cannot encode the implementation identity;
* the restriction is visible before runtime only through non-normative producer information.

Further investigation should still establish the underlying root cause and whether this is representative of a broader class of packages.

---

# Case C — Likepy 0.3.0

## Published release facts

PyPI lists:

```text
Requires-Python: >=3.6,<3.12

likepy-0.3.0.tar.gz

likepy-0.3.0-py3-none-any.whl

Programming Language :: Python :: Implementation :: CPython
```

The project description states:

> Likepy only supports CPython. It does not support PyPy and other Python implementations.

## Existing mechanisms

The Python version restriction is represented by:

```text
Requires-Python: >=3.6,<3.12
```

The implementation restriction is not represented by the wheel:

```text
py3-none-any
```

The classifier communicates the producer's support position descriptively.

## Root-cause classification

**Current classification: F — explicit support-policy declaration.**

The available evidence establishes that the producer declares CPython-only support.

It does not yet establish whether the restriction is:

* runtime-semantic;
* build-related;
* caused by alternative-interpreter limitations;
* due to private implementation usage;
* or simply an explicit project support policy.

This distinction matters because not every support-policy statement should necessarily become a normative installation constraint.

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

    machine-readable positive/negative implementation information
```

But the actual user/installer benefit has not yet been demonstrated.

## Residual status

**Residual candidate — insufficient root-cause evidence for strong classification.**

It remains valuable because it independently reproduces the pattern:

```text
CPython-only producer statement
+
generic Python wheel
+
sdist
+
version restriction represented separately
```

---

# Case D — simple-ctx-log 0.0.3

## Published release facts

PyPI describes a limitation:

```text
Uses sys._getframe (CPython only)
```

The release publishes:

```text
simple_ctx_log-0.0.3.tar.gz
simple_ctx_log-0.0.3-py3-none-any.whl
```

Released:

```text
February 1, 2026
```

## Why it matters

This is a recent independent example of:

```text
pure Python
+
implementation-specific API
+
py3-none-any
+
sdist
```

It is therefore useful for investigating whether implementation support can remain semantically narrower than the artifact tag.

## Root-cause classification

**Current classification: A/F candidate.**

The project identifies `sys._getframe` as the reason for the CPython-only limitation.

However, the current research record has not independently established:

1. whether the relevant API is genuinely unavailable or incompatible on the relevant alternative implementations;
2. whether the package actually fails at runtime there;
3. whether the restriction is a support-policy decision rather than a technical incompatibility.

The source should therefore be inspected before treating this as a strong residual case.

## Existing mechanisms

The wheel remains:

```text
py3-none-any
```

and therefore does not communicate the CPython-only statement.

The implementation distinction also cannot be expressed through the Python version portion of `Requires-Python`.

## Residual status

**Residual candidate — source/root-cause validation pending.**

This case should not yet be counted as a strong residual case.

---

# Case E — TribeCore 4.7.3

## Published release facts

PyPI states:

```text
CPython only.

PyPy and other Python implementations are not supported.
```

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

The project explicitly explains that `py3-none-{platform}` is used so that one wheel can work across Python 3.x versions on a given platform.

## Why it matters

This demonstrates that the phenomenon is not limited to:

```text
py3-none-any
```

The wheel can be platform-specific while remaining implementation-generic:

```text
py3-none-win_amd64
py3-none-manylinux...
py3-none-macosx...
```

Thus:

```text
platform compatibility
```

and:

```text
implementation support
```

can remain separate dimensions.

## Root-cause classification

**Current classification: B/D/F candidate, with H also possible.**

The project uses bundled/native libraries, so the apparent CPython-only boundary may arise from:

* build/toolchain constraints;
* ABI/configuration;
* native components;
* bundled dependencies;
* explicit support policy;
* or a combination.

The release therefore requires deeper artifact and source analysis before it can be used as a clean runtime-support example.

## Resolver consequence

The interesting question is whether the following environment can appear compatible from artifact metadata while violating the producer's stated support boundary:

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

If so, the case may demonstrate a real release-level distinction.

But if the actual restriction is entirely caused by native/ABI/build mechanics already covered elsewhere, the case should not be counted as evidence for a new implementation-support field.

## Residual status

**Residual candidate — important but not yet clean evidence.**

The native/bundled component caveat must be resolved before classification as a strong residual case.

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

The package is a wrapper around platform-specific event-loop implementations.

## Why it is useful

This is primarily a test of **support inheritance**.

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

It is useful primarily as a negative control against incorrectly propagating implementation restrictions through dependencies.

---

# Control cases

## psutil

psutil contains implementation-specific build logic, but supports multiple Python implementations.

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

# Evidence-status summary

| Release              | Producer claim                              | Artifact mismatch                 | Current root cause        | Resolver consequence demonstrated? | Status                  |
| -------------------- | ------------------------------------------- | --------------------------------- | ------------------------- | ---------------------------------- | ----------------------- |
| RestrictedPython 8.5 | CPython only                                | `py3-none-any`                    | F; A pending              | Potentially                        | Candidate               |
| HAX 0.3.0            | CPython only                                | `py3-none-any`                    | A/F                       | Yes, conceptually                  | **Strongest candidate** |
| Likepy 0.3.0         | CPython only                                | `py3-none-any`                    | F pending source analysis | Potentially                        | Candidate               |
| simple-ctx-log 0.0.3 | CPython only                                | `py3-none-any`                    | A/F pending validation    | Potentially                        | Candidate               |
| TribeCore 4.7.3      | CPython only                                | `py3-none-{platform}`             | B/D/F/H possible          | Potentially                        | Candidate               |
| winuvloop 0.2.5      | Indirect implementation-specific dependency | `py3-none-any`                    | H                         | No                                 | Mixed / control         |
| psutil               | Multi-implementation                        | Implementation-specific internals | Not a restriction         | No                                 | Control                 |
| Autobahn             | Multi-implementation                        | Component-level restrictions      | H                         | No                                 | Control                 |
| Guppy3               | CPython/ABI-specific                        | More strongly encoded in wheels   | D                         | Not the target problem             | Control                 |
| bocpy                | CPython/private ABI                         | Version/ABI-sensitive             | D/E                       | Not cleanly                        | Control                 |

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
```

Failure at steps 8–11 means the case remains evidence of a **semantic mismatch**, but not yet evidence that a new standard is necessary.

---

# Important non-inferences

The following conclusions must **not** be drawn from these cases:

### No wheel for an implementation

```text
No PyPy wheel
    ≠
PyPy unsupported
```

A project may simply publish wheels only for the most common implementation while still supporting source installation elsewhere.

### CPython classifier

```text
CPython classifier
    ≠
formal statement that all other implementations are incompatible
```

The classifier is currently treated as descriptive evidence.

### Runtime implementation check

```text
sys.implementation check
    ≠
proof that a new metadata field is required
```

The underlying reason for the check must still be understood.

### Native extension

```text
native extension
    ≠
implementation-support metadata problem
```

The restriction may already belong to wheel tags, ABI information, build requirements, or platform metadata.

### Implementation-specific dependency

```text
dependency supports CPython only
    ≠
top-level package supports CPython only
```

Support can be conditional or provide fallbacks.

### Failed build

```text
one failed build
    ≠
normative unsupported declaration
```

A failed build may result from a temporary toolchain issue, an alternative-interpreter bug, missing external dependency, or other environmental condition.

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

However, the cases do **not yet establish** that a new Core Metadata field is necessary.

The most important unresolved distinction is:

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
```

The current evidence therefore supports the following research position:

> There appears to be a genuine distinction between artifact-level compatibility and producer-declared implementation support. Several releases expose that distinction in practice. The remaining question is whether the distinction is sufficiently well-defined, stable, prevalent, and useful to justify a new normative, machine-actionable metadata field.

The strongest current candidate is **HAX 0.3.0**, because its CPython restriction is explicitly enforced at runtime while its published wheel remains `py3-none-any`.

The other cases should currently be treated as supporting candidates rather than counted as independent proof.

The next investigation should therefore focus on:

1. validating the root cause of each candidate;
2. determining whether the restriction is runtime, build, ABI, dependency, private-API, or support-policy driven;
3. testing whether classifiers are sufficient for the actual consumer;
4. determining whether PEP 725 or existing artifact mechanisms already solve the relevant cases;
5. demonstrating a concrete resolver/index decision that would improve with machine-readable implementation support metadata;
6. measuring whether such cases are common enough to justify a new standard;
7. evaluating stale or overly restrictive producer declarations as a first-class failure mode.

Until those questions are answered, the residual cases should be treated as **research evidence rather than a recommendation for `Supported-Implementation` or `Requires-Implementation`.**
