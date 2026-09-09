# PEP 508 / Dependency Specifier Notes

**URL:** https://packaging.python.org/en/latest/specifications/dependency-specifiers/

**Relevance:** High

## Existing implementation environment markers

The dependency-specifier specification defines environment markers that can conditionally control whether a dependency specification applies to a particular environment.

Among the standardized marker fields are:

```text
platform_python_implementation
implementation_name
implementation_version
```

For example:

```text
Requires-Dist: cffi; implementation_name == "pypy"
```

can express that the dependency on `cffi` applies when the target environment has:

```text
implementation_name == "pypy"
```

The current specification defines:

```text
implementation_name
    -> sys.implementation.name

implementation_version
    -> derived from sys.implementation.version

platform_python_implementation
    -> platform.python_implementation()
```

and provides example values such as `cpython` and `pypy` for `implementation_name`.

## What markers solve

Environment markers answer a dependency-selection question:

```text
Under which environments does this dependency specification apply?
```

Conceptually:

```text
Distribution A
    |
    +-- Requires-Dist: B; implementation_name == "pypy"
```

means that the dependency on `B` is conditional on the target environment.

The current specification states that a marker expression evaluates to either `True` or `False` for a deployment environment. When the expression evaluates to `False`, the dependency is ignored.

This is a mature and standardized mechanism.

## What markers do not directly express

The dependency-marker mechanism does not define a general distribution-level predicate of the form:

```text
This distribution itself is unsupported
when implementation_name == "pypy".
```

The semantic object controlled by an environment marker is the **dependency specification**.

For example:

```text
Requires-Dist: cffi; implementation_name == "pypy"
```

means approximately:

```text
if implementation_name == "pypy":
    require cffi
else:
    do not require cffi
```

It does not mean:

```text
if implementation_name != "pypy":
    reject this distribution
```

That distinction is fundamental.

## Dependency conditionality versus distribution support

The research corpus must therefore distinguish at least these two situations:

```text
implementation-specific dependency
```

and:

```text
implementation-specific release support
```

An implementation-specific dependency can often be represented entirely with existing PEP 508 markers.

For example, a package might require one dependency on CPython and a different dependency on PyPy:

```text
Requires-Dist: dependency-a; implementation_name == "cpython"
Requires-Dist: dependency-b; implementation_name == "pypy"
```

That is a dependency-resolution problem.

It does not necessarily imply that the package itself has an implementation-support restriction.

## A useful counterexample

Consider a distribution that supports both CPython and PyPy but requires different dependencies:

```text
CPython
    -> dependency A

PyPy
    -> dependency B
```

PEP 508 markers are sufficient to describe that conditional dependency structure.

Introducing a `Supported-Implementation` field would add no necessary information merely because implementation identity appears in the dependency metadata.

This is important negative evidence against over-broad use of a proposed support field.

## A different case: the distribution itself is unsupported

Now consider a release that contains code which cannot operate on PyPy at all.

The producer's intended statement might instead be:

```text
This release does not support PyPy.
```

There is no ordinary `Requires-Dist` marker whose defined semantic role is to reject the containing distribution when that marker evaluates to false.

That is a different semantic category from conditional dependency selection.

The question for this research is whether that category needs a standardized release-level representation.

Importantly, this is still only a **semantic distinction**, not proof that a new Core Metadata field is required.

## Why a negative dependency cannot simply substitute

One tempting workaround would be to encode unsupported environments indirectly through dependencies.

For example, a project might attempt to make an unsupported implementation fail by requiring a package that is unavailable there.

This should not be treated as an equivalent representation.

It would turn:

```text
support declaration
```

into:

```text
dependency-resolution side effect
```

and would make the meaning depend on an unrelated distribution's availability.

That approach also cannot reliably communicate the producer's actual support policy to indexes, metadata consumers, search tools, or other packaging systems.

Therefore the existence of PEP 508 markers should not be used to argue that implementation-support metadata can simply be encoded as a deliberately failing dependency.

## Marker evaluation is environment-specific

Environment markers are evaluated against a target environment.

This makes them useful for implementation-aware dependency resolution, but it also means they are fundamentally conditional expressions.

For example:

```text
implementation_name == "pypy"
```

asks:

```text
Does this target environment have implementation name "pypy"?
```

It does not assert:

```text
Does the distribution author support PyPy?
```

The first is an environmental fact.

The second is a producer declaration.

This mirrors the distinction established elsewhere in this research between:

```text
runtime identity
```

and:

```text
release support policy
```

## `implementation_name` and `implementation_version`

The existence of both fields is also relevant to the design question.

A future implementation-support declaration would need to decide whether it concerns only:

```text
implementation_name
```

or also:

```text
implementation_version
```

The current dependency specification already defines version comparisons for `implementation_version`. It derives that value from `sys.implementation.version`.

However, a support declaration should not automatically inherit dependency-marker semantics merely because the same identity information is useful.

For example:

```text
implementation_name == "cpython"
and
implementation_version >= "3.12"
```

could describe a conditional dependency.

Whether the same expression should be permitted as a **release-support declaration** is a separate design question.

## PEP 508 as existing-mechanism evidence

PEP 508 therefore provides a strong existing mechanism for one major class of implementation-specific behavior:

```text
implementation-specific dependency selection
```

Before proposing new metadata, every candidate case should be tested against this mechanism.

The research should ask:

```text
Can the observed implementation difference be solved
by conditional dependencies?
```

If yes, the case should normally be classified as:

```text
existing mechanism sufficient
```

rather than as evidence for a new support field.

If no, the reason should be documented explicitly.

## PEP 508 as a boundary control

PEP 508 establishes an important boundary:

```text
Environment markers
        ↓
conditional dependency semantics
```

A hypothetical release-support field would instead be:

```text
Support metadata
        ↓
producer declaration about the distribution release
```

The two mechanisms may use the same implementation identity vocabulary without having the same semantic role.

This distinction prevents the research from making either of two opposite mistakes:

```text
Mistake A:
"PEP 508 already has implementation markers,
so implementation support metadata is unnecessary."

Mistake B:
"PEP 508 has implementation markers,
so the proposed support field is obviously necessary."
```

Neither conclusion follows automatically.

## Relationship to `Requires-Python`

The same distinction applies to `Requires-Python`.

Conceptually:

```text
Requires-Python
    -> Python language-version compatibility

PEP 508 implementation markers
    -> conditional dependency selection

Wheel tags
    -> built-artifact compatibility

Possible support metadata
    -> producer-declared release-level implementation support
```

These mechanisms can overlap in practical cases, but they do not have identical semantics.

Therefore a residual case should only count as evidence for new metadata after the existing mechanisms have been considered individually.

## Consumer-decision test

For each candidate implementation-specific release, ask:

```text
1. Is the issue actually a dependency-selection problem?

2. If yes, can PEP 508 markers express it?

3. If the issue is not dependency selection,
   what consumer decision is currently impossible?

4. Can wheel tags make that decision for built artifacts?

5. Can Requires-Python make the language-version decision?

6. Can build-system or host requirements express the constraint?

7. If all existing mechanisms are insufficient,
   would release-level implementation metadata
   enable a useful pre-install or pre-build decision?
```

Only a case that survives these questions should remain a serious residual candidate.

## Important warning

The existence of implementation markers does **not** mean that every implementation-specific problem should be expressed through a new support field.

Conversely, the fact that markers cannot reject their containing distribution does **not** prove that such rejection metadata is necessary.

The research must establish actual consumer value.

In particular, a proposed support declaration should not merely duplicate a runtime check such as:

```python
if sys.implementation.name != "cpython":
    raise RuntimeError(...)
```

unless there is a demonstrated benefit from knowing that restriction **before executing or building the package**.

## Current conclusion

PEP 508 is a complementary existing mechanism.

It already provides standardized implementation-aware environment information:

```text
implementation_name
implementation_version
platform_python_implementation
```

and allows dependencies to be conditional on those values.

It does **not**, however, define a general release-level support declaration for the distribution containing the dependency specification.

The research should therefore preserve this distinction:

```text
implementation-specific dependency
        !=
implementation-specific release support
```

The remaining question is whether the second category creates a sufficiently important and recurring packaging problem to justify a separate normative representation in Core Metadata.

PEP 508 should consequently be treated as both:

1. **existing-mechanism evidence**, because many implementation-specific cases are already expressible; and
2. **a semantic boundary**, because conditional dependency selection is not the same thing as declaring the support boundary of the containing distribution.

The existence of PEP 508 therefore narrows the residual problem but does not, by itself, establish or disprove the need for a new release-support field.
