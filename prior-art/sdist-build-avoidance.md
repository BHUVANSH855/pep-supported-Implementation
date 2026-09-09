# Sdist Build Avoidance

## Status

**Relevance:** High

Sdist build avoidance is one of the strongest practical arguments adjacent to the proposed implementation-support metadata.

A resolver may encounter a project release for which no currently compatible wheel is available and may therefore consider the source distribution. Building that sdist can be expensive, slow, or impossible in the current environment.

This creates a genuine packaging problem:

> Can a frontend determine, before attempting an sdist build, that the candidate should not be built?

That problem is related to implementation compatibility, but it is not semantically identical to:

> Does this release support this Python implementation?

The distinction is central to this research.

---

## 1. The basic installation path

A project release may be represented on an index by:

```text
project-version
├── compatible wheel(s)
└── sdist
```

When a compatible wheel is available, an installer can install the wheel without performing a package build.

When no suitable wheel is available, installation tools may fall back to the source distribution and build a wheel from it.

The Python Packaging User Guide describes this general flow: wheels require no compilation step during installation, while tools may fall back to the source distribution when no suitable wheel is available.

Conceptually:

```text
resolve release
      │
      ├── compatible wheel exists
      │       ↓
      │    install wheel
      │
      └── no compatible wheel
              ↓
           obtain sdist
              ↓
           build wheel
              ↓
           install result
```

The problematic path is therefore:

```text
resolve release
      ↓
sdist
      ↓
build
      ↓
failure
```

The failure may occur only after substantial work has already been performed.

---

## 2. Why sdist builds are different from wheel selection

A wheel contains an artifact whose compatibility can be evaluated using its filename and wheel tags.

For example:

```text
package-1.0-cp312-cp312-manylinux_2_17_x86_64.whl
```

provides implementation, ABI, and platform information before installation.

An sdist does not have an equivalent wheel compatibility tag.

The standardized sdist filename is:

```text
{name}-{version}.tar.gz
```

and its archive contains source code together with `pyproject.toml` and Core Metadata.

Therefore:

```text
wheel:
    compatibility can often be evaluated before installation/build

sdist:
    source must potentially be built before the final wheel
    compatibility is known from the resulting artifact
```

This is one reason source distributions are relevant to the proposed metadata question.

---

## 3. Build failure has many possible causes

An sdist build can fail for many reasons.

Possible causes include:

```text
unsupported Python implementation
unsupported Python version
unsupported platform
unsupported ABI
missing compiler
missing linker
missing external library
missing system headers
missing build tool
incorrect build dependency declaration
build backend failure
dependency incompatibility
project policy
private implementation API
implementation-specific semantic assumption
```

These causes have different owners and different potential solutions.

Therefore:

```text
sdist build failed
```

does **not** imply:

```text
release is incompatible with this environment
```

and especially does not imply:

```text
new implementation-support metadata is required
```

The root cause has to be established first.

---

## 4. The three questions that must remain separate

The research should distinguish at least three different predicates.

### A. Should this sdist be built?

```text
No, do not attempt this build.
```

This is a **build-selection / frontend policy** question.

### B. Can this release be installed on this implementation?

```text
No, this release does not support PyPy.
```

This is a **release compatibility/support** question.

### C. Can this particular build environment produce an installable artifact?

```text
The package supports PyPy, but this environment lacks
the compiler or external library needed to build it.
```

This is a **build-environment capability** question.

They can lead to the same observed symptom:

```text
installation fails
```

but they are not the same semantic fact.

---

## 5. Why a generic "do not build" mechanism is different

A frontend could potentially know:

```text
sdist should not be automatically built
```

without knowing:

```text
the project does not support PyPy
```

For example, a project might intentionally require a human or system integrator to build its sdist because:

* it requires a system compiler;
* it requires a large external dependency;
* it has expensive compilation;
* it is intentionally source-only for some environments;
* it requires configuration not appropriate for automatic installation.

In those cases the correct information is about **build policy**, not Python implementation support.

Conversely, a project might explicitly support PyPy but have no PyPy wheel.

Then:

```text
no compatible wheel
```

must not be interpreted as:

```text
PyPy unsupported
```

because the sdist may build successfully.

This is an important reason not to overload an implementation-support field with sdist-build policy.

---

## 6. Direct prior art: preventing unwanted sdist builds

There is particularly relevant Packaging discussion from 2024 titled:

**“Preventing unwanted attempts to build sdists”**

The discussion considered a dedicated metadata mechanism for exactly this practical problem.

Paul Moore proposed a possible field called:

```text
No-SDist-Build
```

with the basic model:

```text
1. Add a metadata field.
2. Expose it through the Simple API, similarly to Requires-Python.
3. Frontends do not select the sdist when the field is present.
4. Provide an explicit override so users can still permit the sdist.
```

The proposed mechanism was explicitly about preventing automatic sdist builds, rather than declaring a general implementation-compatibility property.

Ralf Gommers additionally suggested passing an `allow-sdist` decision through `config_settings` to the build backend so the backend could make build-time decisions based on the user's explicit choice.

This is highly relevant prior art because it demonstrates that the ecosystem has considered a solution at the **sdist-build-policy layer**, independently of implementation-support metadata.

---

## 7. What this prior art proves

The 2024 discussion provides evidence that:

```text
unwanted sdist builds
```

are recognized as a distinct packaging problem.

It does **not** establish that:

```text
No-SDist-Build
```

was standardized.

Nor does it establish that such a mechanism is the preferred current solution.

Its value for this research is architectural:

```text
Problem:
    automatic source build is undesirable

Possible layer:
    sdist/frontend build-selection policy
```

That is different from:

```text
Problem:
    release supports CPython but not PyPy

Possible layer:
    release compatibility metadata
```

The two should not be collapsed merely because both can prevent an attempted build.

---

## 8. PEP 517 already separates build execution from package metadata

PEP 517 defines the standardized interface between build frontends and build backends.

A frontend is responsible for establishing the Python environment in which the backend runs and making the project's declared build requirements available.

This creates another useful distinction:

```text
frontend
    │
    ├── decides which candidate to build
    │
    └── creates build environment
              │
              ↓
          build backend
              │
              ↓
          wheel/sdist
```

A build failure can therefore originate from the build process without implying that the release itself has a simple implementation-support predicate.

For example:

```text
PyPy
  ↓
build backend
  ↓
compiler unavailable
  ↓
build fails
```

does not establish:

```text
PyPy unsupported
```

The failure must be classified.

Source:

* PEP 517: https://peps.python.org/pep-0517/

---

## 9. PEP 725 is an important adjacent mechanism

PEP 725 is especially relevant because it addresses standardized declarations of external dependencies.

It distinguishes:

```text
build dependencies
host dependencies
runtime dependencies
```

and explicitly accounts for cross-compilation.

For example, an sdist may require:

```text
compiler
OpenSSL
libffi
pkg-config
```

or another external component before a wheel can be produced.

A failed build caused by such a missing dependency is fundamentally different from:

```text
this release is CPython-only
```

Therefore a residual-case investigation must ask whether an apparent implementation restriction is actually an undeclared build/host dependency problem.

PEP 725 should consequently be treated as an important alternative/boundary mechanism rather than ignored.

---

## 10. Build dependency failure is not implementation incompatibility

Consider:

```text
package 1.0
supports:
    CPython
    PyPy

build requirement:
    external compiler
```

Environment:

```text
PyPy
compiler absent
```

Result:

```text
sdist build fails
```

It would be incorrect to conclude:

```text
package does not support PyPy
```

The actual predicate is:

```text
package supports PyPy
AND
current build environment lacks a required capability
```

A support field cannot solve that build-environment problem unless it is incorrectly overloaded into a general build capability language.

That would substantially broaden the scope of the proposal.

---

## 11. ABI and platform failures are similarly distinct

Suppose:

```text
package supports CPython
```

but the available sdist requires:

```text
platform-specific library
```

or:

```text
ABI-specific compiler configuration
```

and the local environment cannot satisfy it.

The resulting build failure does not prove:

```text
CPython unsupported
```

Likewise:

```text
no compatible wheel
```

may simply mean:

```text
a wheel has not been published for this platform
```

rather than:

```text
the release does not support this platform
```

This is why the research taxonomy must distinguish:

```text
artifact availability
artifact compatibility
build capability
release support policy
```

---

## 12. The strongest implementation-support residual

The interesting residual is much narrower.

Consider:

```text
project 1.0

sdist:
    available

wheel:
    py3-none-any

Requires-Python:
    >=3.10

producer behavior:
    explicitly rejects non-CPython implementations
```

Suppose the source contains a genuine semantic boundary such as:

```python
if sys.implementation.name != "cpython":
    raise RuntimeError("CPython required")
```

Then:

```text
Requires-Python
    ↓
cannot express CPython-only

py3-none-any
    ↓
does not encode the producer's release-level policy

PEP 508
    ↓
does not express self-support

external build metadata
    ↓
irrelevant if the restriction is semantic rather than a missing dependency
```

This is much stronger evidence for an implementation-support metadata gap.

But it is still necessary to establish that a consumer would actually benefit from knowing this before attempting installation/build.

---

## 13. The generic-wheel problem

This case deserves special attention.

A package can publish:

```text
package-1.0-py3-none-any.whl
```

while source code imposes:

```text
CPython only
```

If that happens, the artifact itself advertises broad compatibility while the producer's actual release behavior is narrower.

That creates a potential mismatch:

```text
artifact compatibility metadata
          ≠
producer support policy
```

This is one of the clearest scenarios in which artifact-level tags and release-level support metadata could potentially have different semantics.

However, the research must establish that the wheel is not simply incorrectly tagged.

If the correct solution is:

```text
publish an implementation-specific wheel tag
```

then a new Core Metadata field would be solving the wrong problem.

Therefore each such case needs an artifact-level audit before entering the residual set.

---

## 14. Source distribution metadata is available before building

Modern sdists contain a `PKG-INFO` file with Core Metadata.

The current sdist specification requires the archive to contain:

```text
pyproject.toml
PKG-INFO
```

and the metadata must conform to at least Core Metadata 2.2.

This is important because it means a release-level metadata field could theoretically be inspected before the build step.

Conceptually:

```text
download sdist
      ↓
read PKG-INFO
      ↓
evaluate release metadata
      ↓
decide whether candidate is appropriate
      ↓
only then build
```

This is technically plausible.

But technical availability does not establish that the metadata is sufficient, authoritative, or worth standardizing.

---

## 15. Static metadata and the sdist/wheel relationship

PEP 643 established important consistency rules for metadata contained in sdists and wheels.

For non-dynamic metadata in an sdist, a wheel built from that sdist generally has to preserve the corresponding metadata value. Current Core Metadata rules continue this model, with additional behavior around dynamic fields in Metadata 2.6.

This matters for a hypothetical implementation-support field.

If the proposed field is:

```text
release-level support policy
```

then it should probably behave similarly to other static release-level metadata:

```text
sdist:
    Supported-Implementation: CPython

wheel built from sdist:
    Supported-Implementation: CPython
```

But that would need to be explicitly specified.

If implementation support can legitimately vary between wheels, the field becomes more complicated and may not belong in Core Metadata at all.

---

## 16. Early metadata is not the same as early candidate rejection

A critical distinction:

```text
metadata is available
```

does not imply:

```text
installer must reject candidate
```

For example, Core Metadata can contain descriptive information that is not an installer constraint.

`Requires-Python` is special because its specification explicitly gives installation tools permission to use it when selecting versions.

A future implementation-support field would therefore need to define:

```text
Is this:
    descriptive?
    advisory?
    a hard compatibility constraint?
    an opt-out build policy?
```

This is one of the most important design decisions in the entire proposal.

---

## 17. Failed-build caching is a different possible solution

Another alternative is to remember failed builds.

Conceptually:

```text
package 1.0
environment fingerprint
        ↓
build attempted
        ↓
failure
        ↓
cache failure
```

A future resolver encountering the same candidate and sufficiently similar environment could avoid immediately repeating the build.

This can solve a practical problem:

```text
Do not repeatedly spend time rebuilding a candidate
that has already failed.
```

But it does not communicate:

```text
The producer declares this release unsupported on PyPy.
```

The distinction is:

```text
metadata:
    producer statement about release

failure cache:
    consumer observation about environment/candidate
```

Therefore failed-build caching is a possible **operational alternative** for repeated failures, but it is not semantically equivalent to support metadata.

It also introduces difficult questions:

* How is the environment fingerprint defined?
* Is a failure deterministic?
* How long should the result be cached?
* Does a compiler update invalidate it?
* Does a dependency update invalidate it?
* Is the failure caused by a temporary upstream bug?
* Can a different build configuration succeed?
* Can another frontend build the same sdist successfully?

These issues make caching useful but fundamentally different from producer-declared support.

---

## 18. Build avoidance versus support declaration

The two concepts can be represented as:

```text
Build avoidance:

    "Do not automatically build this sdist."

                    ↓

        frontend/build policy


Implementation support:

    "This release supports CPython
     but does not support PyPy."

                    ↓

        release compatibility metadata
```

A build-avoidance mechanism might prevent a build even when the release is technically compatible.

A support declaration might reject a release even though the build would technically succeed.

Therefore neither should be used as a semantic substitute for the other without an explicit design argument.

---

## 19. Consumer-benefit test

For a proposed implementation-support field, the strongest consumer benefit would be:

```text
Current environment:
    PyPy 3.12

Candidate:
    package 1.0 sdist

Existing metadata:
    Requires-Python: >=3.10

Current result:
    sdist is considered
        ↓
    build starts
        ↓
    known implementation incompatibility discovered
        ↓
    failure
```

With implementation metadata:

```text
Current environment:
    PyPy 3.12

Candidate metadata:
    supported implementations = {CPython}

Resolver:
    implementation mismatch
        ↓
    candidate rejected
        ↓
    no build
```

The important measurable benefit would therefore be:

```text
avoid a build that is known to be impossible because of
a release-level implementation restriction.
```

That is a stronger use case than simply:

```text
avoid some build failures.
```

The latter can arise from many causes unrelated to implementation support.

---

## 20. Counterexample: support metadata would not solve all sdist failures

Suppose:

```text
package:
    supports CPython

current machine:
    CPython 3.13

sdist:
    requires Rust compiler

machine:
    Rust unavailable
```

Adding:

```text
Supported-Implementation: CPython
```

does nothing.

The resolver would still have to determine:

```text
build environment is insufficient
```

This demonstrates that implementation support metadata cannot become a general pre-build feasibility mechanism.

A proposal that attempts to solve all such cases would need to address a much broader environment-capability model.

That is outside the narrow implementation-support hypothesis.

---

## 21. Counterexample: no wheel does not imply unsupported implementation

Consider:

```text
package 1.0

available artifacts:
    sdist
    cp312 wheel

current interpreter:
    PyPy 3.12
```

There is no PyPy wheel.

It would be incorrect to conclude:

```text
PyPy unsupported
```

The sdist may build successfully on PyPy.

Therefore:

```text
missing implementation-specific wheel
```

must not be counted as evidence of implementation incompatibility without further testing or source evidence.

This is particularly important when constructing the real-world evidence corpus.

---

## 22. Counterexample: failed build does not imply producer policy

Consider:

```text
PyPy
  ↓
sdist build
  ↓
compiler failure
```

Possible interpretations include:

```text
A. PyPy is unsupported.
B. The package is supported, but the compiler is missing.
C. The build backend is broken.
D. A dependency is incompatible.
E. The build environment is incomplete.
F. The project has an implementation-specific bug.
G. The failure is temporary.
```

Only A is direct evidence for implementation-support metadata.

B–G belong to different categories.

This is why the research must preserve root-cause evidence rather than counting failed installations as implementation incompatibility.

---

## 23. Interaction with PEP 725

PEP 725 strengthens this distinction.

It proposes standardized external dependency declarations for:

```text
build-requires
host-requires
dependencies
```

including cross-compilation semantics.

Therefore a case such as:

```text
sdist cannot build because libffi is unavailable
```

should be investigated as an external dependency/build environment case before being classified as implementation incompatibility.

Similarly:

```text
sdist cannot build because the required compiler is absent
```

belongs to build requirements, not necessarily implementation support.

This makes PEP 725 an important exclusion mechanism for the residual-case analysis.

---

## 24. Interaction with PEP 517 build requirements

PEP 517 already defines build isolation and the build backend's Python build environment.

A build frontend must make the project's declared build requirements available to the backend during the relevant hooks.

Consequently, some apparent:

```text
sdist build avoidance
```

cases may actually be:

```text
incorrect or incomplete build-system metadata
```

rather than a missing implementation-support declaration.

This reinforces the requirement that every failure be root-caused.

---

## 25. Interaction with `Requires-Python`

`Requires-Python` can already prevent a release from being selected for an incompatible Python version.

For example:

```text
Requires-Python: >=3.10
```

can prevent a Python 3.9 environment from selecting that release.

But it cannot express:

```text
Python 3.10+
AND
CPython only
```

Therefore a genuine CPython/PyPy release-support distinction can survive the existing version constraint.

This is one reason the implementation-support question remains open.

---

## 26. Interaction with wheel tags

Wheel tags can prevent an incompatible built artifact from being selected.

For example:

```text
cp312-...
```

does not match a PyPy interpreter.

That means a correctly tagged wheel does not require a separate implementation-support field for artifact selection.

The residual problem therefore arises primarily when:

```text
the relevant artifact is generic
```

or:

```text
the candidate is an sdist
```

or:

```text
the restriction is release-level rather than artifact-level.
```

Those cases require separate investigation.

---

## 27. Interaction with PEP 508

PEP 508 provides implementation-aware environment markers for dependencies.

For example:

```text
SomeDependency; implementation_name == "cpython"
```

can express conditional dependency requirements.

But it does not directly state:

```text
this distribution itself is unsupported on PyPy.
```

Therefore a conditional dependency problem should be solved with dependency metadata, while a self-support declaration would be a different semantic category.

---

## 28. The strongest architectural distinction

The research can currently be organized as:

```text
                    Candidate selection
                           │
            ┌──────────────┼───────────────┐
            │              │               │
       Requires-Python   Wheel tags    Implementation
            │              │              support?
            │              │               │
        version         artifact          release
        constraint      compatibility     policy
```

And separately:

```text
                    Build execution
                           │
            ┌──────────────┼───────────────┐
            │              │               │
       PEP 517        PEP 725         Environment
       build API      build/host      capabilities
```

A generic sdist-build-avoidance mechanism would sit closer to:

```text
candidate/build policy
```

than to:

```text
implementation support semantics.
```

This layering should remain explicit throughout the PEP research.

---

## 29. Evidence classification

For this research, sdist-related observations should be classified as follows:

| Observation                                                 | Interpretation             |           Counts as implementation-support evidence? |
| ----------------------------------------------------------- | -------------------------- | ---------------------------------------------------: |
| No wheel exists for PyPy                                    | Artifact availability      |                                                   No |
| Sdist build fails on PyPy                                   | Unknown root cause         |                                                   No |
| Sdist build fails because compiler missing                  | Build environment          |                                                   No |
| Sdist build fails because external library missing          | External dependency        |                                                   No |
| Sdist build fails because ABI unsupported                   | ABI/environment            |                                           Usually no |
| Build backend rejects PyPy explicitly                       | Implementation restriction |                                          Potentially |
| Runtime code explicitly rejects PyPy                        | Implementation restriction |                                          Potentially |
| Project explicitly documents CPython-only semantics         | Producer support policy    |                                          Potentially |
| `py3-none-any` wheel but source explicitly rejects PyPy     | Release/artifact mismatch  |                                 High-value candidate |
| Existing wheel tag already excludes PyPy                    | Artifact compatibility     |                                                   No |
| Conditional dependency handles the difference               | Dependency conditionality  |                                                   No |
| Generic no-build policy would prevent automatic sdist build | Build policy               | No, unless separately tied to implementation support |
| Failed-build cache avoids repeating a known failure         | Consumer observation       |                 Alternative, not semantic equivalent |

---

## 30. Important residual test

A case should enter the implementation-support residual set only if it approximately satisfies all of the following:

```text
1. Exact release is identified.

2. The producer's implementation restriction is established.

3. The restriction is not merely:
       - missing wheel
       - missing compiler
       - missing external dependency
       - platform mismatch
       - ABI mismatch
       - transient build failure
       - dependency problem
       - build backend bug

4. Requires-Python cannot express it.

5. Wheel tags cannot correctly express the relevant release-level fact.

6. PEP 508 conditional dependencies cannot express the required
   self-support semantics.

7. PEP 725/build metadata cannot explain or solve the problem.

8. The relevant restriction applies to the release rather than
   only one artifact.

9. A consumer could make a materially better decision before
   building/installing if the information were available.

10. The information can be made reliable enough to justify
    standardization.
```

This is intentionally a high bar.

---

## 31. Current conclusion

Sdist build avoidance is a real and important packaging problem.

There is direct prior discussion of a possible `No-SDist-Build`-style mechanism intended to prevent frontends from automatically attempting certain source builds.

That prior art demonstrates an important distinction:

```text
"Do not automatically build this sdist"
```

is not equivalent to:

```text
"This release does not support PyPy."
```

The first is a **build-selection/policy** statement.

The second is a **release-level implementation-support** statement.

Likewise:

```text
"this build failed"
```

is not equivalent to either one.

A build may fail because of:

```text
compiler availability
external dependencies
ABI
platform
build backend behavior
dependency resolution
temporary defects
configuration
```

PEP 517, PEP 725, wheel tags, and other existing mechanisms already address parts of these layers.

Therefore sdist-build avoidance should remain an **adjacent problem and an important alternative/design boundary**, rather than being treated as either:

```text
proof that implementation metadata is necessary
```

or:

```text
a complete replacement for implementation-support metadata.
```

The strongest remaining question is narrower:

> Are there real releases where the producer has a genuine release-level implementation-support restriction, the relevant candidate is an sdist or otherwise generically selectable artifact, existing metadata cannot communicate that restriction, and a resolver would materially benefit from knowing it before attempting the build?

That is the residual empirical question the research must answer.

---

## Primary references

* Source Distribution Format
  https://packaging.python.org/en/latest/specifications/source-distribution-format/

* Core Metadata
  https://packaging.python.org/en/latest/specifications/core-metadata/

* PEP 517 — A build-system independent format for source trees
  https://peps.python.org/pep-0517/

* PEP 725 — Specifying external dependencies in pyproject.toml
  https://peps.python.org/pep-0725/

* PEP 643 — Metadata for Package Source Distributions
  https://peps.python.org/pep-0643/

* Python Packaging Discussion — “Preventing unwanted attempts to build sdists”
  https://discuss.python.org/t/preventing-unwanted-attempts-to-build-sdists/54169
