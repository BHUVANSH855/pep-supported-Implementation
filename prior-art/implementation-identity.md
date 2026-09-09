# Python Implementation Identity

## `sys.implementation`

PEP 421 established `sys.implementation` as the standard runtime namespace for identifying the implementation of the currently running Python interpreter.

The required `name` attribute is a lower-case identifier representing the implementation. PEP 421 gives examples including:

```text
cpython
pypy
jython
ironpython
```

The required `version` attribute represents the **implementation version**, rather than the version of the Python language implemented by that runtime. PEP 421 deliberately establishes this distinction because an alternative implementation can implement one Python language version while having a different implementation version.

The current Python documentation preserves these semantics: `sys.implementation.name` is the implementation identifier, and `sys.implementation.version` is the implementation version. The name is guaranteed to be lower case, while the particular string is defined by the implementation.

Source:

https://peps.python.org/pep-0421/

Current runtime documentation:

https://docs.python.org/3/library/sys.html

## Packaging's existing implementation vocabulary

The packaging ecosystem already exposes implementation identity through environment markers.

The current dependency-specifier specification defines:

```text
implementation_name
implementation_version
platform_python_implementation
```

with:

```text
implementation_name
    -> sys.implementation.name

implementation_version
    -> derived from sys.implementation.version

platform_python_implementation
    -> platform.python_implementation()
```

For example, the specification gives `cpython` and `pypy` as sample values for `implementation_name`, and `3.10.12` and `7.3.17` as sample implementation versions for CPython and PyPy respectively.

Source:

https://packaging.python.org/en/latest/specifications/dependency-specifiers/

This is important prior art: a proposed Core Metadata field should not introduce a second, incompatible definition of what a Python implementation is.

## Identity is not support

The existence of a standardized implementation identity does **not** itself establish package support.

These are separate statements:

```text
The current interpreter is CPython.
```

and:

```text
This distribution release supports CPython.
```

The first is an environmental fact.

The second is a producer declaration about a particular distribution release.

This distinction is central to the research.

A package may inspect:

```python
sys.implementation.name
```

because its implementation is technically dependent on CPython. That observation is evidence about the package's behavior and may support a case for an explicit release-level support declaration.

It does not, by itself, establish what semantics such a declaration should have.

Similarly:

```text
implementation_name == "cpython"
```

in a dependency marker is a condition under which a **dependency specification** applies. It is not currently a standard declaration that the package containing that dependency specification supports only CPython. The dependency-specifier specification defines environment markers as conditions controlling whether a dependency specification applies in a particular environment.

Therefore PEP 508 markers are relevant prior art, but they are not themselves a substitute for release-level self-support metadata.

## Implementation version versus Python version

A proposed support declaration must also avoid conflating:

```text
Python language version
```

with:

```text
Python implementation version
```

PEP 421 explicitly distinguishes these concepts.

For CPython they normally coincide, but they need not coincide for alternative implementations. PEP 421 uses PyPy as the motivating example: its implementation version and the Python language version it implements can be different.

This matters because the existing:

```text
Requires-Python
```

field expresses Python-version compatibility, while a hypothetical implementation-support declaration would operate on a different dimension.

A future specification must therefore avoid semantics such as:

```text
implementation = CPython 3.12
```

being interpreted ambiguously as either:

```text
CPython implementation version 3.12
```

or:

```text
Python language version 3.12 running on CPython
```

Those are related but conceptually distinct.

## Exact identity versus compatibility

The original research question identified an important ambiguity:

```text
Does "cpython" mean exactly:

    sys.implementation.name == "cpython"

or does it mean:

    CPython-compatible runtime?
```

For machine-actionable metadata, the first interpretation is substantially more deterministic.

PEP 421 defines `sys.implementation.name` as the implementation identifier and constrains it to a lower-case identifier, but it does not define the identifier as a declaration of behavioral compatibility with another implementation.

Consequently, a metadata specification should not silently turn:

```text
cpython
```

into a broader concept such as:

```text
any runtime sufficiently compatible with CPython
```

unless that compatibility relation is separately standardized.

Otherwise a resolver could not determine whether a declaration applies merely from the runtime's implementation identity.

## CPython-derived and compatible runtimes

This becomes particularly important for CPython-derived runtimes and other environments that may intentionally provide substantial CPython compatibility.

There are at least three potentially different claims:

```text
A. The release was tested on CPython.

B. The release supports interpreters whose
   sys.implementation.name is "cpython".

C. The release supports runtimes that are
   behaviorally compatible with CPython.
```

These claims should not be treated as equivalent.

A producer may be able to make A without being willing to make B as a normative compatibility claim.

Likewise, B does not automatically imply C.

This is another reason to avoid defining a support field around informal labels such as:

```text
CPython
PyPy
GraalPy
```

without first specifying the machine-readable identity and matching semantics.

## Relationship to wheel tags

Implementation identity also must not be confused with artifact compatibility.

Wheel tags already encode implementation information at the artifact level. For example, the Python tag can distinguish implementation-specific tags such as:

```text
cp
pp
```

from the generic:

```text
py
```

This answers an artifact-level question:

```text
Can this particular wheel artifact be considered compatible
with the target interpreter?
```

A release-level support declaration would answer a different question:

```text
Does the producer claim that this distribution release
supports this Python implementation?
```

Therefore `sys.implementation` should be treated as part of the conceptual vocabulary underlying both mechanisms, rather than as evidence that wheel tags or a hypothetical support field are interchangeable.

## Relationship to implementation markers

The existing environment-marker vocabulary provides useful prior art for any future support metadata.

In particular:

```text
implementation_name
implementation_version
```

already have defined meanings and are evaluated against the target environment. The current specification also defines how `implementation_version` is derived from `sys.implementation.version`.

A future Core Metadata field should therefore either:

1. reuse these established semantics directly, or
2. provide a strong justification for introducing a different identity model.

Inventing a second implementation-name vocabulary would create unnecessary interoperability risk.

However, reusing the vocabulary does **not** mean that the field should simply embed an arbitrary PEP 508 marker expression. The semantic question remains whether the proposed metadata is:

```text
a declarative support boundary
```

rather than:

```text
a conditional dependency expression.
```

## What PEP 421 does not establish

PEP 421 establishes runtime implementation identity.

It does not establish:

* package support policy;
* distribution metadata semantics;
* resolver eligibility rules;
* whether a package is installable on an implementation;
* whether a package author promises support for an implementation;
* whether compatibility means tested support, technical compatibility, or a guarantee;
* how a distribution should declare a set of supported implementations.

This boundary is important for the current research.

`sys.implementation` can provide the identity against which a support declaration could be evaluated, but it does not itself provide the missing support declaration.

## Research implications

The implementation-identity question should therefore be separated into two layers.

### Layer 1: Runtime identity

The ecosystem already has this:

```text
sys.implementation.name
sys.implementation.version
```

and corresponding packaging environment markers:

```text
implementation_name
implementation_version
```

These are established mechanisms.

### Layer 2: Distribution support declaration

The unresolved question is whether Core Metadata needs a release-level statement equivalent to:

```text
Supported-Implementation: cpython
```

and, if so, what exactly that statement means.

That second question cannot be answered merely by pointing to `sys.implementation`.

## Open semantic questions

Before defining any implementation-support metadata, research must resolve at least:

```text
1. Is the value matched against sys.implementation.name?

2. If implementation versions are supported, are they matched
   against sys.implementation.version?

3. Does the declaration describe:
   - tested support,
   - technical compatibility,
   - producer support policy,
   - or a normative installation guarantee?

4. Does absence of the field mean:
   - no declaration,
   - all implementations,
   - or unknown support?

5. Is support an allow-list, deny-list, or more general expression?

6. Can a release support different implementation sets
   depending on Python language version?

7. How are CPython-derived or compatibility runtimes treated?

8. How does the field interact with wheel tags, Requires-Python,
   environment markers, and source builds?

9. What resolver decision would become possible that existing
   mechanisms cannot make?

10. What happens when the declaration is stale or incorrect?
```

These questions are semantic requirements, not merely syntax questions.

## Current conclusion

Python already has a standardized and machine-readable implementation identity through `sys.implementation`, and packaging already exposes that identity through `implementation_name` and `implementation_version` environment markers.

Therefore the research should **not invent a second implementation identity vocabulary**.

The unresolved problem is narrower:

```text
runtime implementation identity
        !=
distribution release support declaration
```

If a future Core Metadata field is justified, its implementation identity should preferably be defined in terms of the existing ecosystem vocabulary rather than introducing a new meaning for values such as `CPython` or `PyPy`.

In particular, a declaration such as:

```text
Supported-Implementation: CPython
```

must have an explicit machine-actionable matching rule. The strongest initial candidate is exact implementation identity corresponding to:

```python
sys.implementation.name == "cpython"
```

rather than an undefined notion of "CPython-compatible".

This is a **design constraint**, not evidence that a new field is necessary.

The research must still establish that an actual consumer decision remains unsolved after existing mechanisms—including `Requires-Python`, wheel tags, dependency markers, build/host metadata, and source-build behavior—have been considered.
