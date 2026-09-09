# `Requires-Python` Prior Art

**Current specification:** Core Metadata 2.6
**Authoring counterpart:** `[project].requires-python` in `pyproject.toml`
**Original standardization:** PEP 345 / Core Metadata 1.2
**Primary source:** https://packaging.python.org/en/latest/specifications/core-metadata/
**Related sources:** PEP 345, dependency specifiers, platform compatibility tags

## Status

**Relevance:** Critical

`Requires-Python` is the closest existing Core Metadata mechanism to the proposed implementation-support concept. It provides a normative, machine-readable declaration that a distribution is compatible with particular Python versions, and installation tools may use it when selecting project versions.

The key research question is therefore not simply:

> Can `Requires-Python` express implementation support?

It cannot.

The more important question is:

> Does the fact that `Requires-Python` is version-oriented establish a genuine metadata gap for implementation support that warrants a new Core Metadata field?

That question remains open.

---

## 1. Normative semantics

The current Core Metadata specification defines:

> `Requires-Python` specifies the Python version(s) that the distribution is compatible with.

Its value must use the standard Python version-specifier syntax.

For example:

```text
Requires-Python: >=3.10
```

The specification explicitly states that installation tools may use this field when choosing which version of a project to install.

The authoring equivalent in `pyproject.toml` is:

```toml
[project]
requires-python = ">=3.10"
```

The `pyproject.toml` specification maps this directly to the Core Metadata `Requires-Python` field.

This makes `Requires-Python` materially different from descriptive metadata such as Trove classifiers: it is an actual compatibility constraint that installers can use.

Sources:

* Core Metadata 2.6: https://packaging.python.org/en/latest/specifications/core-metadata/
* `pyproject.toml` specification: https://packaging.python.org/en/latest/specifications/pyproject-toml/

---

## 2. Historical origin

`Requires-Python` was introduced by PEP 345 as part of Core Metadata 1.2.

PEP 345 described the field as specifying the Python version(s) that a distribution is **guaranteed to be compatible with**.

Examples included:

```text
Requires-Python: 2.5
Requires-Python: >2.1
Requires-Python: >=2.3.4
Requires-Python: >=2.5,<2.7
```

The field was therefore designed from the beginning around the **Python version dimension**, rather than around implementation identity.

This historical design matters because it shows that the distinction between Python-version compatibility and other compatibility dimensions is not a recent packaging limitation. `Requires-Python` was deliberately defined as a version requirement.

Source:

* PEP 345: https://peps.python.org/pep-0345/

---

## 3. What `Requires-Python` actually solves

The semantic dimension can be represented as:

```text
Python version compatibility
        │
        └── Requires-Python
```

Examples:

```text
Requires-Python: >=3.10
```

or:

```text
Requires-Python: >=3.10,<3.14
```

This allows an installer to reject or skip a release whose declared Python-version compatibility does not include the current interpreter version.

For example:

```text
Current Python:
    3.9

Release:
    Requires-Python: >=3.10

Result:
    release is incompatible
```

This is an important example of **producer-declared release compatibility becoming an installer-visible constraint**.

That is precisely the property a proposed implementation-support field would need to justify independently.

---

## 4. What it does not express

`Requires-Python` does not identify the Python implementation.

For example:

```text
Requires-Python: >=3.10
```

does not distinguish:

```text
CPython 3.10
PyPy 3.10
GraalPy 3.10
other Python implementations
```

Therefore the following policy cannot be expressed directly:

```text
CPython 3.10+
```

while excluding:

```text
PyPy 3.10+
GraalPy 3.10+
```

Likewise, it cannot directly express:

```text
CPython 3.10+
PyPy 3.10+
```

while excluding another implementation.

This is a genuine semantic limitation.

However:

> A semantic limitation is not by itself evidence that a new Core Metadata field is required.

The research must establish that the missing information is both useful and not better represented by an existing packaging layer.

---

## 5. Python version and implementation are independent dimensions

The compatibility space is better represented as:

```text
                    Python version
                         │
                         │
             ┌───────────┴───────────┐
             │                       │
        3.10 / 3.11 / ...      implementation
                                     │
                        ┌────────────┼────────────┐
                        │            │            │
                     CPython       PyPy        GraalPy
```

`Requires-Python` describes the first dimension.

It does not describe the second.

This distinction is important because implementation identity can matter even when the Python language version is identical.

For example, a package may:

```text
support:
    CPython 3.12

not support:
    PyPy 3.12
```

while still legitimately declaring:

```text
Requires-Python: >=3.12
```

if the author intends that field to describe the supported Python-version range rather than the complete implementation policy.

Whether such a release should additionally carry an implementation-support declaration is the central unresolved question.

---

## 6. `Requires-Python` is not equivalent to implementation support

A tempting interpretation would be:

```text
Requires-Python
        +
implementation identity
        =
complete Python compatibility declaration
```

That should not yet be assumed.

Compatibility can involve multiple independent dimensions:

```text
Python version
implementation
ABI
platform
external libraries
build requirements
runtime dependencies
CPU features
operating-system features
private implementation APIs
security guarantees
```

Existing packaging standards already divide these dimensions among different mechanisms.

For example:

| Dimension                                  | Existing mechanism                    |
| ------------------------------------------ | ------------------------------------- |
| Python version                             | `Requires-Python`                     |
| Dependency conditionality                  | PEP 508 environment markers           |
| Wheel implementation/version compatibility | compatibility tags                    |
| ABI                                        | wheel ABI tags and related mechanisms |
| Platform                                   | wheel platform tags                   |
| External/build requirements                | PEP 725 mechanisms                    |
| Environment features                       | mechanisms such as PEP 780            |
| Descriptive support information            | Trove classifiers                     |

Therefore the existence of a missing implementation dimension does not automatically imply that it belongs next to `Requires-Python`.

---

## 7. Important distinction: requirement versus support declaration

There is also a semantic question about the proposed field's name.

`Requires-Python` is a **requirement**.

For example:

```text
Requires-Python: >=3.10
```

can naturally be interpreted as:

> Installation requires a Python version satisfying this constraint.

A hypothetical:

```text
Requires-Implementation: CPython
```

would similarly suggest:

> Installation requires CPython.

But a package maintainer may instead want to communicate:

```text
Supported-Implementation:
    CPython
    PyPy
```

meaning:

> These implementations are explicitly supported by this release.

Those statements are not necessarily equivalent.

Consider a package which has been tested only on CPython but has no known CPython-specific code:

```text
tested:
    CPython

unknown:
    PyPy
    GraalPy
```

It would be difficult to infer from that situation that PyPy is a requirement violation.

Conversely, a package containing:

```python
if sys.implementation.name != "cpython":
    raise RuntimeError(...)
```

has a much stronger implementation boundary.

Therefore the research must not assume that the existing `Requires-*` naming model is automatically the correct semantic model.

This is one reason `Supported-Implementation` and `Requires-Implementation` should remain separate hypotheses until the real-world cases are classified.

---

## 8. `Requires-Python` cannot simply use an environment marker

The Core Metadata specification explicitly states that `Requires-Python` cannot be followed by an environment marker.

Therefore this is not a valid way to extend the field:

```text
Requires-Python: >=3.10; implementation_name == "cpython"
```

The standard dependency marker machinery instead exists in dependency specifications.

PEP 508 defines markers including:

```text
implementation_name
implementation_version
platform_python_implementation
```

These can be used to make a dependency conditional.

For example:

```text
SomeDependency; implementation_name == "cpython"
```

But this mechanism controls whether a **dependency requirement applies**.

It does not mean:

> Reject this distribution itself unless the current interpreter is CPython.

That semantic distinction is critical.

Source:

* Dependency specifiers: https://packaging.python.org/en/latest/specifications/dependency-specifiers/

---

## 9. Why PEP 508 does not automatically solve this

PEP 508 gives packaging tools standardized implementation identity information.

That makes it possible to express:

```text
dependency X is needed on CPython
```

or:

```text
dependency Y is needed on PyPy
```

It does not provide a standard way to say:

```text
this distribution release supports only CPython
```

Those are different predicates.

Conceptually:

```text
PEP 508:

    IF environment == CPython
        THEN require dependency X


Hypothetical implementation-support metadata:

    distribution release
        supports CPython
        does not support PyPy
```

The former is conditional dependency semantics.

The latter would be release compatibility/support semantics.

Therefore PEP 508 is an important boundary, but not direct evidence that a new field is needed.

A real-world case that can be solved entirely through conditional dependencies should not be counted as a residual case for this proposal.

---

## 10. Relationship to wheel compatibility tags

Wheel compatibility tags already have implementation-aware semantics.

The platform compatibility tag specification defines:

```text
{python tag}-{abi tag}-{platform tag}
```

The Python tag identifies the implementation and version required by a built distribution.

Examples include:

```text
cp312
pp312
py3
```

where:

```text
cp = CPython
pp = PyPy
py = generic Python
```

The specification explicitly states that the Python tag indicates the implementation and version required by a distribution.

Therefore a wheel can already communicate implementation-specific artifact compatibility.

For example:

```text
package-1.0-cp312-none-manylinux_2_17_x86_64.whl
```

has a materially different implementation compatibility meaning from:

```text
package-1.0-py3-none-any.whl
```

Sources:

* Platform compatibility tags: https://packaging.python.org/en/latest/specifications/platform-compatibility-tags/
* Binary distribution format: https://packaging.python.org/en/latest/specifications/binary-distribution-format/

---

## 11. The important `py3-none-any` case

This creates a particularly important boundary for the research.

A wheel such as:

```text
py3-none-any
```

is intentionally generic.

The compatibility-tag specification describes `py3` as a generic Python implementation tag rather than CPython-specific compatibility.

Therefore:

```text
py3-none-any
```

can be appropriate for a pure-Python distribution that is genuinely implementation-independent.

But it does not necessarily prove that the producer has positively verified every Python implementation.

This produces an important possible residual case:

```text
artifact:
    py3-none-any

release behavior:
    explicitly CPython-only
```

If the source code contains an explicit CPython-only restriction, the wheel tag may fail to communicate the producer's actual release-level support policy.

That is potentially relevant to a new release-level metadata field.

However, this still requires investigation of the exact case.

A generic wheel tag does not itself prove that a new Core Metadata field is the right solution. The restriction might instead be:

* a runtime failure;
* a build restriction;
* a dependency restriction;
* a private CPython API dependency;
* an ABI issue;
* an artifact-tagging error;
* or an intentional producer support policy.

Those cases must be separated.

---

## 12. Core Metadata versus artifact metadata

The wheel specification contains an especially important architectural distinction.

The wheel compatibility tag is part of the **built distribution's metadata**, not the Core Metadata `METADATA` file.

The PEP 425 model explains that:

> `METADATA` / `PKG-INFO` should be valid for an entire distribution, not a single build.

This gives us a useful design boundary:

```text
Core Metadata
    ↓
release/distribution-level facts


Wheel compatibility tags
    ↓
specific artifact compatibility
```

This distinction is directly relevant to implementation support.

If:

```text
release version 1.0
```

has a single implementation-support policy regardless of artifact, Core Metadata could plausibly be the right layer.

If instead:

```text
wheel A → CPython only
wheel B → PyPy only
wheel C → generic Python
```

then artifact compatibility mechanisms are more natural.

The proposed field must therefore not duplicate information that belongs in wheel tags.

---

## 13. Source distributions make the question harder

Wheel tags cannot solve every implementation-support question because source distributions do not use wheel compatibility tags.

An sdist contains Core Metadata in its `PKG-INFO`.

The current source-distribution specification requires an sdist to contain:

```text
pyproject.toml
PKG-INFO
```

and the metadata must conform to at least Core Metadata 2.2.

This means release-level metadata can be present in an sdist before the package is built.

Source:

* Source distribution format: https://packaging.python.org/en/latest/specifications/source-distribution-format/

This is potentially important for the proposed feature because one possible motivation is:

```text
Current environment:
    PyPy

Available release:
    source distribution

Without implementation metadata:
    download/build
        ↓
    discover incompatibility

With implementation metadata:
    inspect metadata
        ↓
    reject/skip before build
```

That is a legitimate potential consumer benefit.

But it is not yet sufficient evidence.

The research must determine whether existing build metadata, dependency metadata, resolver behavior, or failed-build caching can solve the same practical problem.

---

## 14. Metadata availability versus metadata semantics

The existence of `Requires-Python` in an sdist demonstrates that compatibility information can be available before building the project.

However:

```text
metadata exists early
```

does not automatically imply:

```text
installer is required to use every metadata field for candidate rejection
```

This distinction matters for a hypothetical implementation-support field.

Even if we standardize:

```text
Supported-Implementation: CPython
```

we would still need to define:

1. how installers interpret it;
2. whether it is a hard compatibility constraint;
3. whether absence means unknown or unrestricted;
4. whether it applies equally to sdists and wheels;
5. whether the value is release-level or artifact-level;
6. how it interacts with wheel tags;
7. how it interacts with source builds;
8. how it interacts with conditional dependencies;
9. what happens when a declaration becomes stale.

Therefore the existence of `Requires-Python` is prior art for **machine-readable release compatibility metadata**, but not a complete design template for implementation support.

---

## 15. Why classifiers are not equivalent to `Requires-Python`

Trove classifiers commonly describe supported Python versions and implementations.

For example:

```text
Programming Language :: Python :: Implementation :: CPython
```

can communicate CPython support to users and indexes.

However, the packaging documentation explicitly distinguishes classifiers from `requires-python`.

Classifiers are used for searching and browsing and are not the mechanism for restricting installation based on Python version.

The documentation states that to actually restrict what Python versions a project can be installed on, `requires-python` should be used.

This is important prior art:

```text
Classifier
    ↓
descriptive/discoverability metadata


Requires-Python
    ↓
installer-visible compatibility constraint
```

The same distinction should not automatically be assumed for implementation classifiers, but it provides a strong precedent for asking whether implementation classifiers are intentionally descriptive rather than normative.

Source:

https://packaging.python.org/en/latest/guides/writing-pyproject-toml/

---

## 16. The central insufficiency

The strongest statement we can make at this stage is:

> `Requires-Python` cannot represent implementation identity or implementation-specific release support.

For example, it cannot directly distinguish:

```text
Release A:
    CPython 3.10+
    PyPy 3.10+

Release B:
    CPython 3.10+
    not PyPy
```

if both have:

```text
Requires-Python: >=3.10
```

This is a real representational gap.

But:

> Representational gap ≠ demonstrated need for a new field.

The missing information could belong to another mechanism, or the practical cases may be sufficiently rare that the cost of introducing and adopting new metadata outweighs the benefit.

---

## 17. What a new field would add beyond `Requires-Python`

A hypothetical implementation-support field would add an independent compatibility dimension.

Conceptually:

```text
Requires-Python:
    version constraint

Implementation field:
    implementation constraint/support declaration
```

For example:

```text
Requires-Python: >=3.10
Supported-Implementation: CPython
```

would communicate something that the first field cannot.

However, this creates several design questions:

### 17.1 Intersection semantics

Would the installer interpret:

```text
Requires-Python: >=3.10
Supported-Implementation: CPython
```

as:

```text
Python version >= 3.10
AND
implementation == CPython
```

Presumably yes, but this would need normative definition.

### 17.2 Multiple implementations

Would:

```text
Supported-Implementation: CPython
Supported-Implementation: PyPy
```

mean logical OR?

If so, the field becomes a set-valued compatibility declaration.

### 17.3 Absence

What does this mean?

```text
Requires-Python: >=3.10
```

with no implementation field?

Possible interpretations include:

```text
all implementations
```

or:

```text
implementation support unspecified
```

or:

```text
no implementation restriction declared
```

These are materially different.

The safest semantic default would likely be **unspecified**, rather than silently meaning universal support.

But this is a design hypothesis, not an established standard.

### 17.4 Partial testing

What should a package do when:

```text
CPython: tested
PyPy: untested
GraalPy: untested
```

Is the correct declaration:

```text
CPython only
```

or:

```text
CPython explicitly supported
```

or:

```text
no declaration
```

This question strongly favors a distinction between **support** and **requirement**.

---

## 18. `Requires-Python` is a useful analogy, not proof of symmetry

It is tempting to argue:

```text
We have Requires-Python.
Therefore we should have Requires-Implementation.
```

That inference is too strong.

The two dimensions have different existing packaging infrastructure.

Python version compatibility has long been directly expressed through:

```text
Requires-Python
```

while implementation compatibility is already partially represented through:

```text
wheel Python tags
PEP 508 implementation markers
sys.implementation
Trove classifiers
```

Therefore the burden of proof for a new implementation field is higher.

The research needs to demonstrate a **residual class of cases** that survives those existing mechanisms.

---

## 19. Required residual-case test

A real-world example should not be counted as evidence for a new implementation-support field merely because it says:

```text
CPython only
```

The case should be examined through the following sequence:

```text
1. Exact release identified
        ↓
2. What does the producer actually declare?
        ↓
3. Is the restriction runtime, build-time, ABI, dependency,
   private-API, security, or explicit support policy?
        ↓
4. Can Requires-Python express it?
        ↓
5. Can wheel compatibility tags express it?
        ↓
6. Can conditional dependencies express it?
        ↓
7. Can build/host dependency metadata express it?
        ↓
8. Is the artifact generic even though the release is restricted?
        ↓
9. Is there an sdist case where the restriction is invisible
   before building?
        ↓
10. Would a resolver actually make a better decision using
    implementation metadata?
        ↓
11. Is that benefit large enough to justify new metadata?
```

Only cases surviving this process should enter the residual evidence set.

---

## 20. Important negative cases

The following should **not** automatically be treated as evidence that `Requires-Python` is insufficient:

### No PyPy wheel

```text
No pp* wheel exists.
```

This does not prove that the release does not support PyPy.

A source distribution or generic wheel may still work.

### CPython classifier

```text
Programming Language :: Python :: Implementation :: CPython
```

does not by itself prove that all other implementations are unsupported.

### `sys.implementation` usage

A package inspecting:

```python
sys.implementation
```

does not automatically establish a metadata requirement.

The code may be performing optional behavior, diagnostics, feature detection, or fallback handling.

### Native extension

A native extension does not automatically imply:

```text
CPython only
```

because compatibility may be represented through wheel Python/ABI tags or stable ABI mechanisms.

### Source-build failure

A failed PyPy source build does not automatically establish producer-declared incompatibility.

The failure may instead be:

```text
missing compiler
missing system dependency
broken build backend
dependency issue
unsupported build environment
temporary bug
ABI problem
```

These are different problems.

---

## 21. Strongest potential residual

The strongest theoretical residual case looks approximately like this:

```text
Release:
    project X 1.0

Artifact:
    py3-none-any wheel
    and/or sdist

Requires-Python:
    >=3.10

Actual release behavior:
    explicitly rejects non-CPython implementations

Reason:
    known implementation-specific semantic restriction

Existing mechanisms:
    Requires-Python       → insufficient
    wheel tag             → artifact is generic
    dependency markers    → not the package's own support constraint
    ABI tags              → not the issue
    build requirements    → not the issue
    external dependencies → not the issue

Consumer problem:
    resolver/install tool cannot determine incompatibility
    before installation or source build

Potential benefit:
    avoid an otherwise doomed candidate
```

This is the type of evidence required to make a serious case for new metadata.

The existence of such cases must be demonstrated empirically rather than assumed.

---

## 22. Relationship to PEP 508 implementation markers

PEP 508's implementation markers make implementation identity available to dependency resolution.

Relevant marker concepts include:

```text
implementation_name
implementation_version
platform_python_implementation
```

This means the packaging ecosystem already has a standardized vocabulary for identifying implementations.

That is valuable prior art for any future proposal.

However, the semantics are different:

```text
PEP 508:
    "When should this dependency apply?"

Requires-Python:
    "Which Python versions is this distribution compatible with?"

Hypothetical implementation support:
    "Which Python implementations does this release support/require?"
```

A future proposal should reuse the existing implementation vocabulary rather than inventing a second incompatible naming system, if a new field is eventually justified.

---

## 23. Relationship to source-build avoidance

One of the strongest arguments for a new implementation field would be avoiding a known-bad source build.

For example:

```text
resolver
    ↓
finds sdist
    ↓
downloads source
    ↓
creates build environment
    ↓
builds package
    ↓
package rejects PyPy
```

If the release had reliable implementation-support metadata:

```text
resolver
    ↓
reads release metadata
    ↓
implementation mismatch
    ↓
skip candidate
```

That could be a real optimization and correctness improvement.

However, this argument must be tested against alternatives such as:

* build/host dependency metadata;
* richer artifact compatibility;
* resolver failure caching;
* persistent knowledge of failed builds;
* improved build diagnostics;
* producer-specific metadata;
* or simply fixing incorrect generic wheel/source packaging.

The existence of a costly failure does not establish that a new Core Metadata field is the optimal solution.

---

## 24. Release-level versus artifact-level semantics

`Requires-Python` describes compatibility at the distribution metadata level.

Wheel tags, by contrast, describe individual built artifacts.

This distinction is important for implementation support.

If:

```text
project 1.0
```

has a consistent policy:

```text
CPython only
```

across its sdist and all wheels, a release-level declaration could be coherent.

But if:

```text
project 1.0
    ├── cp311 wheel
    ├── pp311 wheel
    └── py3 wheel
```

have different compatibility properties, the problem may be artifact selection rather than release-level support.

The proposal must therefore define whether implementation support is intended to mean:

```text
property of the release
```

or:

```text
property of an artifact
```

`Requires-Python` is useful prior art for the former, while wheel tags are stronger prior art for the latter.

---

## 25. Current evidence assessment

| Question                                                               | Finding                   | Confidence |
| ---------------------------------------------------------------------- | ------------------------- | ---------: |
| Does `Requires-Python` express Python version compatibility?           | Yes                       |       High |
| Can it express CPython-only support?                                   | No                        |       High |
| Can it express CPython + PyPy support as an implementation set?        | No                        |       High |
| Can it use PEP 508 environment markers?                                | No                        |       High |
| Can PEP 508 markers express implementation identity?                   | Yes                       |       High |
| Do PEP 508 markers express the distribution's own support policy?      | No                        |       High |
| Can wheel tags express implementation-specific artifact compatibility? | Yes                       |       High |
| Can wheel tags describe an sdist's release-level support policy?       | No                        |       High |
| Are classifiers equivalent to installer compatibility constraints?     | No                        |       High |
| Does the limitation prove a new Core Metadata field is necessary?      | No                        |       High |
| Is implementation support a potentially distinct metadata dimension?   | Yes                       |       High |
| Is a new field's exact semantics established?                          | No                        |       High |
| Is there a demonstrated residual consumer need?                        | Still under investigation |          — |

---

## 26. Research conclusion

`Requires-Python` establishes an important precedent:

> Core Metadata can contain a machine-readable, release-level compatibility constraint that installation tools may use when selecting project versions.

It also establishes a clear boundary:

> `Requires-Python` describes Python version compatibility, not Python implementation identity.

Therefore:

```text
Requires-Python
        ≠
implementation support
```

is well established.

However, the correct conclusion is **not**:

> Therefore `Requires-Implementation` is required.

The stronger and more defensible conclusion is:

> `Requires-Python` demonstrates that release-level compatibility declarations can be normative Core Metadata, but implementation compatibility must still justify its own metadata layer because wheel tags, PEP 508 implementation markers, classifiers, build metadata, and other mechanisms already cover adjacent parts of the problem.

The research should therefore continue by identifying real releases where:

```text
implementation support matters
        AND
existing mechanisms cannot represent the relevant fact
        AND
a consumer can make a materially better pre-install/pre-build decision
        AND
the information is genuinely a release-level property.
```

Until such residual cases are demonstrated, `Requires-Python` should be treated as **strong structural prior art and a clearly identified semantic gap, not as proof that a new implementation-support field is necessary**.

---

## Primary references

* Core Metadata 2.6
  https://packaging.python.org/en/latest/specifications/core-metadata/

* PEP 345 — Metadata for Python Software Packages 1.2
  https://peps.python.org/pep-0345/

* Dependency Specifiers / PEP 508
  https://packaging.python.org/en/latest/specifications/dependency-specifiers/

* Platform Compatibility Tags / PEP 425
  https://packaging.python.org/en/latest/specifications/platform-compatibility-tags/

* Binary Distribution Format / Wheel
  https://packaging.python.org/en/latest/specifications/binary-distribution-format/

* `pyproject.toml` specification
  https://packaging.python.org/en/latest/specifications/pyproject-toml/

* Writing `pyproject.toml` / project metadata guidance
  https://packaging.python.org/en/latest/guides/writing-pyproject-toml/

* Source Distribution Format
  https://packaging.python.org/en/latest/specifications/source-distribution-format/
