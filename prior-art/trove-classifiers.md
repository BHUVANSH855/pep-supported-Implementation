# Trove Classifier Prior Art

## Status

**Relevance:** Critical

Trove classifiers are the strongest existing counterargument to introducing a new implementation-support field.

The Python packaging ecosystem already has standardized classifier vocabulary for Python implementations, including:

```text
Programming Language :: Python :: Implementation :: CPython
Programming Language :: Python :: Implementation :: PyPy
Programming Language :: Python :: Implementation :: GraalPy
Programming Language :: Python :: Implementation :: IronPython
Programming Language :: Python :: Implementation :: Jython
Programming Language :: Python :: Implementation :: MicroPython
Programming Language :: Python :: Implementation :: Stackless
```

Therefore the ecosystem already has a vocabulary for communicating implementation support.

The central question is not:

> Can Python packaging identify implementations?

It can.

The question is:

> Is descriptive implementation classification sufficient, or is there a demonstrated need for a separate **normative implementation-compatibility constraint** that installers use during candidate selection?

That distinction is the key issue.

---

## 1. Existing implementation vocabulary

The Python Package Index classifier vocabulary includes implementation-specific classifiers such as:

```text
Programming Language :: Python :: Implementation :: CPython
Programming Language :: Python :: Implementation :: PyPy
```

and classifiers for other Python implementations.

This provides an established ecosystem vocabulary for implementation identity/support.

The current `pyproject.toml` specification maps:

```toml
[project]
classifiers = [...]
```

directly to the Core Metadata `Classifier` field.

Core Metadata defines `Classifier` as a multiple-use field in which each entry gives a classification value for the distribution.

This means a package can already publish:

```text
Classifier: Programming Language :: Python :: Implementation :: CPython
```

without inventing a new vocabulary.

---

## 2. What classifiers are designed to do

The current packaging documentation describes classifiers as information about the project that can be used for classification and discovery.

For example:

```text
Development Status :: 5 - Production/Stable
Programming Language :: Python :: 3
Programming Language :: Python :: Implementation :: CPython
Topic :: ...
```

The `Classifier` Core Metadata field itself is fundamentally a classification mechanism rather than a general-purpose installation constraint.

The current packaging guide makes the distinction especially explicit for Python-version classifiers:

> classifiers are used for searching and browsing projects on PyPI, not for installing projects.

For actual Python-version installation restrictions, the documentation directs authors to `requires-python`.

The important existing model is therefore:

```text
Classifier
    ↓
description / classification / discovery


Requires-Python
    ↓
normative installation compatibility
```

The proposed implementation field would effectively introduce:

```text
Implementation classifier
    ↓
currently descriptive


new implementation metadata
    ↓
normative installation compatibility
```

That is the semantic change that must be justified.

---

## 3. The January 2024 implementation-metadata discussion

There is particularly relevant prior discussion from January 2024 titled:

**“Python implementation in metadata”**

The original question asked why Python implementation information was not stored in project metadata and whether a package could require a particular implementation, with PyPy given as the motivating example.

Paul Moore questioned why a library would need to block users of other Python implementations and suggested that, where implementation support needs to be communicated, classifiers were the appropriate mechanism.

His argument was that classifiers:

```text
declare explicitly what implementations
the project is willing to support
```

without forcing installation behavior.

The discussion therefore established a useful existing philosophy:

```text
implementation support declaration
        ↓
classifier
        ↓
do not automatically impose an installation restriction
```

This is a significant counterargument to the current proposal.

---

## 4. What the 2024 discussion did and did not establish

The January 2024 discussion should not be treated as a definitive rejection of normative implementation metadata.

It considered use cases such as:

```text
development environment
application deployment
library support
```

and focused substantially on communicating which implementations a project is willing to support.

The newer research question is narrower:

```text
release-level implementation compatibility
+
installer candidate selection
+
especially source distributions
```

The distinction is important.

The 2024 discussion effectively asked:

> How should a project communicate which implementations it supports?

The current research asks:

> Is there a need for a machine-readable implementation constraint that causes an installer to reject a release candidate before installation/build?

Those are related but not identical questions.

---

## 5. The strongest argument against a new field

The strongest counterargument can be stated simply:

```text
The ecosystem already has implementation classifiers.
```

Therefore, if the intended meaning is:

```text
"We support CPython."
```

then:

```text
Classifier:
    Programming Language :: Python :: Implementation :: CPython
```

already communicates that fact.

Introducing:

```text
Supported-Implementation: CPython
```

would initially appear redundant.

The proposal therefore needs to identify information that classifiers cannot communicate **and that consumers genuinely need**.

---

## 6. Support versus exclusion

This is the most important semantic distinction.

A classifier:

```text
Programming Language :: Python :: Implementation :: CPython
```

does not necessarily mean:

```text
PyPy is unsupported.
```

It can instead mean:

```text
CPython is an implementation the project explicitly identifies
as supported.
```

The absence of:

```text
Programming Language :: Python :: Implementation :: PyPy
```

does not necessarily establish:

```text
PyPy is incompatible.
```

Therefore classifiers naturally support an **open-world / descriptive** interpretation:

```text
listed implementation
    → explicitly classified/supported

unlisted implementation
    → unspecified
```

A normative compatibility field might instead need:

```text
listed implementation
    → compatible

unlisted implementation
    → incompatible
```

Those semantics are substantially different.

---

## 7. Why absence is the critical issue

Consider:

```text
Classifier:
    Programming Language :: Python :: Implementation :: CPython
```

What does that mean?

Possible interpretations include:

```text
A. CPython is supported; others unknown.
B. CPython is supported; others unsupported.
C. CPython is preferred; others may work.
D. CPython is the only tested implementation.
E. CPython is the only implementation the author is willing to support.
```

The current classifier system does not define the field as a hard compatibility predicate equivalent to:

```text
if implementation != CPython:
    reject
```

That is precisely why classifiers cannot simply be treated as `Requires-Implementation`.

---

## 8. The distinction from `Requires-Python`

The difference becomes clearer when comparing implementation classifiers with `Requires-Python`.

For Python versions:

```text
Requires-Python: >=3.10
```

has normative compatibility semantics.

For implementation:

```text
Classifier: Programming Language :: Python :: Implementation :: CPython
```

does not currently have equivalent rejection semantics.

The resulting model is:

```text
Python version:
    classifier
        +
    Requires-Python

Implementation:
    classifier
        +
    wheel tags
        +
    PEP 508 markers
        +
    no release-level normative constraint yet
```

The proposed field would fill that last gap only if the research proves that such a gap is practically important.

---

## 9. Why classifiers are still useful even without normative semantics

It would be a mistake to conclude that classifiers are ineffective simply because installers do not use them as hard constraints.

They provide:

* PyPI presentation;
* search and filtering;
* documentation to users;
* project classification;
* ecosystem statistics;
* a standardized vocabulary;
* human-readable support information.

For example:

```text
Programming Language :: Python :: Implementation :: PyPy
```

is meaningful evidence that the project explicitly identifies PyPy.

It can help a user decide whether to try the package.

The question is therefore not:

```text
Are classifiers useful?
```

They clearly are.

The question is:

```text
Is the additional consequence of candidate rejection
worth introducing a separate normative field?
```

---

## 10. The current PyPA guidance is an important precedent

The current packaging guide explicitly says that classifiers are often used to declare the Python versions a project supports, but that this information is used for searching and browsing rather than installation.

For actual installation restrictions, the guide says to use:

```text
requires-python
```

instead.

This establishes a strong design principle:

```text
descriptive support information
        ≠
installer compatibility constraint
```

If implementation support follows the same model, then:

```text
Implementation classifier
```

would remain descriptive while a separate normative mechanism would be necessary only if implementation compatibility needs hard candidate filtering.

This makes the proposal's burden of proof quite precise.

---

## 11. Could classifiers simply be given normative semantics?

One possible alternative to a new field is:

> Change the semantics of implementation classifiers so that they become installation constraints.

This would avoid introducing a new vocabulary.

For example:

```text
Classifier:
    Programming Language :: Python :: Implementation :: CPython
```

could theoretically mean:

```text
supported implementations = {CPython}
```

and therefore cause a PyPy candidate to be rejected.

However, this would be a major semantic change.

Existing packages already use classifiers for descriptive purposes.

A classifier's absence currently does not necessarily mean incompatibility.

Changing that interpretation would risk turning incomplete metadata into hard negative compatibility declarations.

For example:

```text
package has:
    CPython classifier

package lacks:
    PyPy classifier
```

does not necessarily mean:

```text
PyPy is forbidden.
```

Therefore simply changing classifier semantics could create substantial compatibility and correctness problems.

---

## 12. Closed-world versus open-world interpretation

This can be expressed formally.

### Descriptive classifier model

```text
classifier contains CPython
        ↓
CPython explicitly identified/supported

classifier does not contain PyPy
        ↓
PyPy support unspecified
```

This is approximately:

```text
open world
```

### Normative support-list model

```text
Supported-Implementation: CPython
        ↓
CPython supported
        ↓
PyPy not in supported set
        ↓
PyPy candidate rejected
```

This is approximately:

```text
closed world
```

A proposal must decide which semantics are actually desired.

This is one reason the names:

```text
Requires-Implementation
```

and:

```text
Supported-Implementation
```

are not interchangeable.

---

## 13. `Requires-Implementation` may be more naturally exclusionary

Suppose a package declares:

```text
Requires-Implementation: CPython
```

The semantics naturally suggest:

```text
current implementation MUST satisfy this requirement
```

That fits candidate rejection.

By contrast:

```text
Supported-Implementation: CPython
```

naturally suggests:

```text
CPython is explicitly supported
```

but leaves open whether an unlisted implementation is:

```text
unsupported
unknown
untested
or merely not explicitly declared
```

This ambiguity is not present in ordinary requirement syntax.

Therefore the classifier comparison raises a deeper question:

> Is the actual desired feature an implementation **requirement**, or an implementation **support declaration**?

That question should be resolved from real-world use cases rather than from the proposed field name.

---

## 14. Implementation classifiers do not encode version constraints

Another limitation is that implementation classifiers identify implementations but do not provide the same expressive version-specifier semantics as `Requires-Python`.

For example, the desired policy might eventually be:

```text
CPython >=3.12
PyPy >=3.10
```

or:

```text
CPython >=3.12,<3.15
```

A classifier such as:

```text
Programming Language :: Python :: Implementation :: CPython
```

does not by itself express those implementation-version constraints.

However, this should **not** automatically be used as evidence for a new field.

It may be sufficient for the actual use case to express:

```text
implementation identity
```

while Python version remains expressed through:

```text
Requires-Python
```

Or the implementation-version problem may belong to another mechanism.

The research should therefore avoid designing a general implementation-version constraint language until real cases require it.

---

## 15. Relationship to PEP 508

PEP 508 already standardizes implementation identity for environment markers.

Relevant markers include:

```text
implementation_name
implementation_version
platform_python_implementation
```

Therefore the ecosystem already possesses machine-readable implementation identity.

This creates another strong counterargument to a new field:

```text
implementation identity already exists
```

through:

```text
sys.implementation
        ↓
PEP 508 environment markers
```

However, PEP 508 markers apply to dependency specifications.

They do not mean:

```text
this distribution itself is incompatible with PyPy.
```

Thus:

```text
PEP 508
    → conditional dependency behavior

implementation classifier
    → descriptive support information

hypothetical implementation metadata
    → release compatibility
```

These remain distinct semantic layers.

---

## 16. Relationship to wheel tags

Wheel tags provide another implementation-aware mechanism.

For example:

```text
cp312
pp312
py3
```

distinguish:

```text
CPython
PyPy
generic Python
```

at the artifact-selection layer.

Therefore a package publishing:

```text
cp312-...
```

already gives installers implementation-specific artifact information.

This weakens the case for a new field for binary artifact compatibility.

The stronger residual case is instead:

```text
sdist
+
generic wheel
+
release-level implementation restriction
```

where the restriction is not appropriately represented by the artifact tag.

---

## 17. The generic-wheel counterexample

Consider:

```text
Classifier:
    Programming Language :: Python :: Implementation :: CPython

Wheel:
    py3-none-any
```

This is not automatically contradictory.

The classifier may simply mean:

```text
CPython explicitly supported
```

while the generic wheel may mean:

```text
artifact has no implementation-specific
wheel compatibility restriction
```

If the package genuinely runs on PyPy, everything is consistent.

If the package actually fails on PyPy, there may be a metadata problem.

The classifier alone does not determine which situation exists.

Therefore real-world research must inspect:

```text
source
release behavior
wheel tags
documentation
tests
build configuration
```

rather than treating the classifier as proof of incompatibility.

---

## 18. The strongest classifier-based residual case

The most interesting case would look like:

```text
Release:
    project 1.0

Classifier:
    CPython

Wheel:
    py3-none-any

Requires-Python:
    >=3.10

Source:
    explicitly rejects non-CPython

Sdist:
    available
```

Here:

```text
classifier
    → communicates CPython support

Requires-Python
    → communicates Python version

wheel tag
    → generic artifact

source behavior
    → establishes actual CPython-only restriction
```

The remaining question becomes:

> Is there a consumer-facing reason that the producer's negative implementation boundary must be machine-readable as a hard candidate constraint?

If yes, this is potentially strong evidence for a new field.

If no, classifiers may remain sufficient.

---

## 19. A classifier is not a negative declaration

This principle should be explicit in the research:

```text
CPython classifier
        ≠
"not PyPy"
```

Similarly:

```text
PyPy classifier
        ≠
"not CPython"
```

The classifier vocabulary is naturally suited to positive classification.

A normative compatibility set would require explicit semantics for:

```text
positive support
negative support
unknown support
untested support
```

The current classifier system does not provide those distinctions.

---

## 20. Why a second field could nevertheless be justified

A new field could be justified if the proposal demonstrates that the ecosystem needs a distinction between:

```text
"CPython is a supported implementation"
```

and:

```text
"this release must not be installed on non-CPython implementations."
```

The first can be communicated by:

```text
Classifier: ... :: CPython
```

The second requires a normative compatibility predicate.

That is the strongest possible justification for a second representation.

But it must be demonstrated through actual consumer behavior.

The proposal should not assume that every CPython-only project needs installer enforcement.

---

## 21. Consumer decision test

For every candidate real-world case, ask:

### What can the classifier tell the consumer?

```text
CPython is explicitly identified.
```

### What decision cannot the classifier support?

Potentially:

```text
reject candidate on PyPy
```

### Does the consumer actually need that decision?

For example:

```text
resolver:
    PyPy
candidate:
    sdist
classifier:
    CPython
```

Would the resolver benefit from:

```text
reject before build
```

or is:

```text
try build
```

still the correct behavior because the classifier is only descriptive?

This is the critical empirical question.

---

## 22. Stale and incomplete classifiers

Another major issue is classifier accuracy.

Projects can:

* add implementation classifiers;
* forget to remove old classifiers;
* stop testing an implementation;
* begin supporting an implementation without updating classifiers;
* publish generic wheels despite implementation-specific code;
* change implementation support between releases.

Therefore a normative installer constraint would place considerably greater correctness requirements on the metadata than ordinary classification.

For example:

```text
Classifier:
    CPython
```

might remain unchanged for years even if PyPy becomes supported.

If the classifier were interpreted as:

```text
only CPython is compatible
```

that stale metadata would actively prevent installation.

This is a strong argument for preserving the current descriptive semantics rather than silently promoting classifiers to hard constraints.

---

## 23. Project-level versus release-level classifiers

Another subtle issue is scope.

The current classifier mechanism is commonly presented as project metadata:

```toml
[project]
classifiers = [...]
```

but Core Metadata is attached to a specific distribution/release.

A future implementation-support field would need to define whether it means:

```text
this release supports CPython
```

or:

```text
this project generally supports CPython
```

The distinction matters when support changes between versions.

For example:

```text
1.0:
    CPython only

2.0:
    CPython + PyPy
```

A release-level metadata field could represent this cleanly.

A project-wide interpretation would not.

This is another reason not to assume that classifiers and normative support metadata have identical semantics.

---

## 24. Interaction with `Requires-Python`

A package may currently publish:

```text
Requires-Python: >=3.10
Classifier: Programming Language :: Python :: 3
Classifier: Programming Language :: Python :: Implementation :: CPython
```

The first is installer-visible compatibility.

The latter two are classification.

This creates an established two-layer model:

```text
version compatibility
    → normative

implementation classification
    → descriptive
```

A new field would effectively add:

```text
implementation compatibility
    → normative
```

The proposal therefore needs to explain why the implementation dimension deserves a normative mechanism when the existing classifier mechanism was intentionally kept descriptive.

---

## 25. Interaction with source distributions

The strongest argument for making implementation metadata normative is probably the sdist case.

Suppose:

```text
project 1.0
    ├── sdist
    └── py3-none-any wheel

Requires-Python: >=3.10

Classifier:
    CPython

actual source:
    CPython-only
```

An installer operating on PyPy may see no artifact-level reason to reject the release.

The classifier communicates:

```text
CPython is supported
```

but does not normatively communicate:

```text
PyPy is forbidden.
```

If the resolver could obtain Core Metadata before downloading/building the sdist, a dedicated normative field could theoretically allow:

```text
candidate rejection
```

before an otherwise doomed build.

This is the strongest case where classifiers may be insufficient.

But it remains a hypothesis until demonstrated with real releases and actual consumer behavior.

---

## 26. The 2026 distinction

The newer 2026 pre-PEP discussion explicitly revisits this issue.

It notes that the January 2024 classifier discussion was about a different problem: using metadata to communicate or enforce an application's development/runtime implementation choice.

The newer proposal instead focuses on:

```text
normative runtime compatibility
+
installer candidate selection
+
especially sdists
```

and asks whether that deserves Core Metadata semantics.

This is important because it means the 2024 discussion should be treated as:

```text
important counterargument
```

rather than:

```text
definitive rejection of the current proposal.
```

---

## 27. Possible alternatives to a second field

The research should explicitly compare at least these options:

### Option A — Keep classifiers descriptive

```text
Classifier:
    ... :: CPython
```

Meaning:

```text
CPython explicitly supported/classified.
```

No hard candidate rejection.

### Option B — Give implementation classifiers normative semantics

```text
Classifier:
    ... :: CPython
```

Meaning:

```text
only CPython is supported.
```

This risks breaking the existing positive/descriptive semantics.

### Option C — Add a new normative field

For example:

```text
Requires-Implementation: CPython
```

This preserves classifiers as descriptive metadata.

### Option D — Add a support field

For example:

```text
Supported-Implementation: CPython
```

This introduces explicit release-level support semantics.

### Option E — No new metadata

Rely on:

```text
classifiers
+
wheel tags
+
Requires-Python
+
PEP 508
+
runtime/build behavior
```

This remains a valid possible conclusion.

---

## 28. Decision criteria

The new field should not be justified merely because:

```text
classifiers are not installer constraints.
```

That establishes only a semantic difference.

The stronger test is:

```text
A. There are real releases with an implementation-specific
   release-level restriction.

B. The restriction matters before installation/build.

C. Existing wheel tags cannot represent the relevant fact.

D. Requires-Python cannot represent it.

E. PEP 508 markers cannot represent it as a self-support predicate.

F. Build/host metadata cannot represent the actual problem.

G. Classifiers communicate insufficiently strong semantics.

H. A consumer would make a materially better decision
   if the normative fact were available.

I. The producer can state the fact reliably at release time.
```

Only when these conditions hold does the classifier limitation become evidence for a new field.

---

## 29. Current evidence assessment

| Question                                                                           | Finding                   | Confidence |
| ---------------------------------------------------------------------------------- | ------------------------- | ---------: |
| Does the ecosystem have implementation classifiers?                                | Yes                       |       High |
| Are CPython/PyPy represented?                                                      | Yes                       |       High |
| Are other implementations represented?                                             | Yes                       |       High |
| Are classifiers part of Core Metadata?                                             | Yes                       |       High |
| Are classifiers available through `[project].classifiers`?                         | Yes                       |       High |
| Are classifiers primarily classification/discovery metadata?                       | Yes                       |       High |
| Does current guidance say classifiers are not Python-version install restrictions? | Yes                       |       High |
| Does a CPython classifier mean all other implementations are incompatible?         | No established semantics  |       High |
| Can classifiers express implementation identity?                                   | Yes                       |       High |
| Can they express a normative self-support requirement?                             | Not currently             |       High |
| Would changing classifier semantics be risky?                                      | Yes                       |       High |
| Is a separate field therefore automatically necessary?                             | No                        |       High |
| Is there a plausible residual case where classifiers are insufficient?             | Yes                       |       High |
| Has that residual case been demonstrated at sufficient scale?                      | Still under investigation |          — |

---

## 30. Current conclusion

Trove classifiers provide strong existing prior art for implementation support vocabulary.

They already allow a project to communicate:

```text
Programming Language :: Python :: Implementation :: CPython
```

or:

```text
Programming Language :: Python :: Implementation :: PyPy
```

and similar implementation classifications.

The current ecosystem intentionally treats classifiers as descriptive/search metadata rather than hard installation constraints. The packaging guide explicitly directs projects to `requires-python` when they need an actual Python-version installation restriction.

The January 2024 Packaging discussion went further and argued that classifiers are appropriate when a project wants to communicate which implementations it is willing to support without forcing an implementation-specific installation requirement.

This is a serious counterargument to a new field.

However, it does not conclusively answer the narrower 2026 question.

The key distinction is:

```text
Classifier:
    "CPython is an implementation this project supports."

Normative implementation metadata:
    "This release is incompatible with PyPy."
```

The first does not necessarily imply the second.

Therefore the proposal should **not** argue:

> Classifiers cannot be used for candidate filtering, therefore a new field is necessary.

That skips the critical empirical step.

Instead, it should argue only if the evidence supports it:

> Existing classifiers provide the vocabulary and descriptive support signal, but there are demonstrated release-level cases where consumers need a stronger negative compatibility predicate before installation or source building, and that predicate cannot be represented correctly by existing mechanisms.

If those cases cannot be demonstrated convincingly, then retaining classifiers as the ecosystem's implementation-support mechanism remains a legitimate and potentially preferable outcome.

The research should therefore treat classifiers as the **primary alternative to a new field**, not merely as obsolete prior art.

---

## Primary references

* Core Metadata 2.6
  https://packaging.python.org/en/latest/specifications/core-metadata/

* `pyproject.toml` specification
  https://packaging.python.org/en/latest/specifications/pyproject-toml/

* Writing `pyproject.toml`
  https://packaging.python.org/en/latest/guides/writing-pyproject-toml/

* Python implementation in metadata — January 2024 discussion
  https://discuss.python.org/t/python-implementation-in-metadata/42653

* Pre-PEP: Requires-Implementation — September 2026 discussion
  https://discuss.python.org/t/pre-pep-requires-implementation-declaring-python-implementation-compatibility-in-core-metadata/108898
