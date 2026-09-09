# `Supported-Platform` Historical Core Metadata

## Status

**Current specification:** Core Metadata 2.6
**Relevance:** Medium

`Supported-Platform` is an existing Core Metadata field that superficially resembles the proposed implementation-support concept.

It should therefore be examined before introducing another compatibility-related field.

The conclusion is clear:

> `Supported-Platform` should not be repurposed to declare Python implementation support.

Its historical semantics concern the operating system and CPU for which a binary distribution was compiled, and the Core Metadata specification explicitly leaves the field's semantics unspecified.

---

## 1. Current specification

Core Metadata 2.6 defines `Supported-Platform` as a multiple-use field.

The specification states that binary distributions containing `PKG-INFO` use the field to specify:

```text
OS
CPU
```

for which the binary distribution was compiled.

The specification explicitly says:

> The semantics of the `Supported-Platform` field are not specified.

Historical examples include:

```text
Supported-Platform: RedHat 7.2
Supported-Platform: i386-win32-2791
```

Source:

https://packaging.python.org/en/latest/specifications/core-metadata/

---

## 2. Historical role

`Supported-Platform` dates to Core Metadata 1.1.

Its historical role is therefore tied to an older packaging model in which binary distribution metadata could carry a free-form platform description.

The field's subject is:

```text
operating system
+
CPU
```

rather than:

```text
Python implementation
```

This distinction matters.

A Python implementation is something such as:

```text
CPython
PyPy
GraalPy
IronPython
Jython
```

whereas a supported platform is something closer to:

```text
Linux
Windows
macOS
x86
ARM
Red Hat
```

These are different compatibility dimensions.

---

## 3. Why the name is misleading for this proposal

The word:

```text
Supported
```

makes the field superficially attractive.

A proposed value such as:

```text
Supported-Platform: CPython
```

might appear plausible to someone reading only the field name.

But this would conflict with the field's established subject.

The current specification expects the field to describe:

```text
OS / CPU
```

not interpreter identity.

Therefore:

```text
Supported-Platform: CPython
```

would introduce a new interpretation unrelated to the field's documented historical purpose.

---

## 4. Its semantics are intentionally underspecified

The most important problem is not merely the field name.

The Core Metadata specification explicitly says:

```text
the semantics of Supported-Platform
are not specified
```

This means consumers cannot rely on a standardized compatibility predicate equivalent to:

```text
if current_platform != Supported-Platform:
    reject
```

There is no standardized language defining:

* matching rules;
* normalization;
* version semantics;
* multiple-value semantics;
* resolver behavior;
* artifact versus release scope;
* or failure behavior.

Therefore `Supported-Platform` is not a suitable existing compatibility constraint that can simply be extended to implementation identity.

---

## 5. Comparison with wheel platform tags

Modern wheel compatibility is handled much more precisely by wheel tags.

A wheel filename can contain:

```text
python tag
-
ABI tag
-
platform tag
```

For example:

```text
cp312-cp312-manylinux_2_17_x86_64
```

The platform component is machine-readable and participates directly in artifact selection.

Therefore:

```text
Supported-Platform
```

should not be treated as the modern equivalent of:

```text
wheel platform tag
```

The former is historical and semantically unspecified.

The latter is part of the standardized wheel compatibility mechanism.

---

## 6. Comparison with `Requires-Python`

The distinction from `Requires-Python` is even clearer.

`Requires-Python` has explicit normative semantics:

```text
Python version compatibility
```

and installation tools may use it when choosing project versions.

`Supported-Platform` instead has:

```text
historical OS/CPU subject
+
unspecified semantics
```

Therefore the two fields should not be treated as equivalent compatibility constraints.

Conceptually:

```text
Requires-Python
    ↓
normative Python-version compatibility


Supported-Platform
    ↓
historical binary platform description
```

---

## 7. Why repurposing it would be problematic

Repurposing the field would create several compatibility problems.

### 7.1 Existing meaning

Existing producers may use the field to describe operating-system or CPU information.

Changing its meaning would make old metadata ambiguous.

### 7.2 Existing consumers

A consumer expecting:

```text
OS / CPU
```

could encounter:

```text
CPython
```

and have no standardized interpretation.

### 7.3 Mixed values

A future producer might want to communicate:

```text
Supported-Platform: Linux
Supported-Platform: CPython
```

Under a repurposed interpretation, it would become unclear whether:

```text
Supported-Platform
```

contains:

```text
platform values
implementation values
or both.
```

### 7.4 Lack of matching rules

Even if CPython were accepted as a value, the specification would still need to define how consumers match it against the current interpreter.

That would effectively require creating a new semantic specification on top of a historically underspecified field.

---

## 8. Implementation identity is not a platform

The distinction can be represented as:

```text
                    Compatibility
                         │
            ┌────────────┴─────────────┐
            │                          │
        Platform                  Interpreter
            │                          │
      OS / CPU                  implementation
            │                          │
     Linux / ARM                CPython / PyPy
```

A single installation can simultaneously have:

```text
platform:
    Linux / x86_64

implementation:
    CPython 3.12
```

These are independent dimensions.

Therefore a metadata field whose subject is platform should not be overloaded to represent implementation identity.

---

## 9. Relationship to wheel tags

Wheel tags already provide a much stronger mechanism for artifact-level platform compatibility.

For example:

```text
manylinux_2_17_x86_64
```

communicates a platform constraint at the artifact level.

The Python tag separately communicates implementation/version compatibility:

```text
cp312
pp312
py3
```

This is a useful demonstration that Python packaging already treats:

```text
implementation
```

and:

```text
platform
```

as separate dimensions.

A new release-level implementation-support mechanism should preserve that distinction.

---

## 10. Relationship to `Platform`

Core Metadata also contains a separate:

```text
Platform
```

field.

The current specification describes `Platform` as a platform specification describing an operating system supported by the distribution that is not listed in the relevant Trove classifiers.

This reinforces the historical division:

```text
Platform
Supported-Platform
    ↓
OS / platform-related metadata


Requires-Python
    ↓
Python version


wheel Python tag
    ↓
Python implementation/version for artifact compatibility
```

There is therefore no strong reason to reinterpret `Supported-Platform` as a Python implementation field.

---

## 11. Relationship to Core Metadata's general design

Core Metadata contains fields serving different purposes.

For example:

```text
Name
Version
Requires-Python
Requires-Dist
Classifier
Project-URL
Supported-Platform
```

They do not all have identical consumer semantics.

Some are:

```text
identity
```

some are:

```text
compatibility constraints
```

some are:

```text
dependency declarations
```

and some are:

```text
descriptive metadata
```

Therefore the fact that `Supported-Platform` exists does not imply that another compatibility dimension should be inserted into it.

A new implementation-support concept should have explicitly defined semantics rather than inheriting ambiguity from this historical field.

---

## 12. Multiple-use behavior does not solve the problem

`Supported-Platform` is a multiple-use field.

That might superficially suggest it could represent:

```text
Supported-Platform: CPython
Supported-Platform: PyPy
```

But multiple-use syntax is not enough.

The standard would still need to define:

```text
what the values mean
```

and:

```text
how consumers interpret them.
```

A field being syntactically capable of carrying multiple strings does not make it semantically suitable for a new compatibility dimension.

---

## 13. Historical examples demonstrate the intended layer

The specification's historical examples:

```text
Supported-Platform: RedHat 7.2
Supported-Platform: i386-win32-2791
```

clearly point toward:

```text
OS
CPU
platform/build target
```

rather than:

```text
Python implementation
```

This makes repurposing especially difficult to justify.

A new field would be clearer and safer if implementation support is ultimately proven to require release-level metadata.

---

## 14. Could it be deprecated instead?

A separate question is whether the ecosystem should eventually deprecate or replace `Supported-Platform`.

That may be reasonable as independent packaging work, but it is outside the current proposal.

The implementation-support research should not depend on:

```text
deprecate Supported-Platform
```

or:

```text
reuse Supported-Platform
```

unless evidence shows that such a change is actually necessary.

The current investigation only establishes:

> The existing field is not a suitable semantic home for Python implementation support.

---

## 15. Comparison with proposed implementation support

The desired concept is closer to:

```text
Supported-Implementation:
    CPython
    PyPy
```

or possibly:

```text
Requires-Implementation:
    CPython
```

than to:

```text
Supported-Platform
```

The important difference is the subject:

| Field/concept                       | Subject                                      |
| ----------------------------------- | -------------------------------------------- |
| `Requires-Python`                   | Python version                               |
| wheel Python tag                    | Python implementation/version of an artifact |
| wheel platform tag                  | OS/platform of an artifact                   |
| `Supported-Platform`                | Historical OS/CPU metadata                   |
| hypothetical implementation support | Python implementation support of a release   |

This table demonstrates that the proposed concept occupies a potentially distinct semantic dimension.

It does **not**, however, establish that a new field is necessary.

---

## 16. Stronger alternative: use existing artifact metadata where appropriate

If a real-world implementation restriction is actually:

```text
this particular binary artifact requires CPython
```

then wheel tags may already provide the correct representation.

For example:

```text
cp312-...
```

is more precise than attempting to encode the same fact in:

```text
Supported-Platform
```

Therefore every real-world case should first be checked at the artifact layer.

Only a restriction that remains:

```text
release-level
+
implementation-specific
+
not correctly expressible through artifact compatibility
```

should contribute to the residual evidence for a new Core Metadata field.

---

## 17. Stronger alternative: use dependency markers where appropriate

If the implementation distinction is actually:

```text
dependency A is needed on CPython
dependency B is needed on PyPy
```

then PEP 508 implementation markers are the appropriate mechanism.

For example:

```text
Requires-Dist: dependency-a; implementation_name == "cpython"
```

This is fundamentally different from declaring:

```text
this distribution itself is unsupported on PyPy
```

Therefore the existing dependency-marker mechanism should be exhausted before treating a case as evidence for a new implementation-support field.

---

## 18. Current evidence assessment

| Question                                                               | Finding | Confidence |
| ---------------------------------------------------------------------- | ------- | ---------: |
| Does `Supported-Platform` exist in current Core Metadata?              | Yes     |       High |
| Is it multiple-use?                                                    | Yes     |       High |
| Does its documented subject include OS/CPU?                            | Yes     |       High |
| Are its semantics specified as a compatibility predicate?              | No      |       High |
| Is it intended to identify Python implementations?                     | No      |       High |
| Would `CPython` fit its documented subject?                            | No      |       High |
| Could repurposing create ambiguity for existing consumers?             | Yes     |       High |
| Do wheel tags already provide platform/artifact compatibility?         | Yes     |       High |
| Should the field be silently reused for implementation support?        | No      |       High |
| Does the field's existence prove a new implementation field is needed? | No      |       High |

---

## 19. Current conclusion

`Supported-Platform` is useful prior art primarily as a **boundary case**.

It demonstrates that Core Metadata has historically contained platform-related fields, but it does not provide an existing standardized mechanism for Python implementation compatibility.

The current specification states that:

```text
Supported-Platform
```

describes the OS and CPU for which a binary distribution was compiled, while explicitly leaving its semantics unspecified.

Therefore:

```text
Supported-Platform
        ≠
Python implementation support
```

and reusing it would create ambiguous semantics and compatibility risks.

The research should consequently keep the proposed concept separate:

```text
platform compatibility
        ≠
Python implementation compatibility
```

However, this is **not evidence that a new Core Metadata field is necessary**.

The correct next step remains empirical:

```text
find real release
    ↓
identify actual implementation restriction
    ↓
test wheel tags
    ↓
test Requires-Python
    ↓
test dependency markers
    ↓
test build/host metadata
    ↓
determine whether the remaining fact is genuinely
release-level implementation support
    ↓
determine whether a consumer needs it before installation/build
```

Only then can the research establish whether a distinct implementation-support field is justified.
