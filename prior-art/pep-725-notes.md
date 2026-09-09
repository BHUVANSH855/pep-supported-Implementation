# PEP 725 Notes — External Dependencies

**URL:** https://peps.python.org/pep-0725/

**Relevance:** High

## What PEP 725 addresses

PEP 725 proposes standardized metadata for dependencies on external systems and components.

Its central distinction is between different dependency contexts, including:

```text id="7n4m2x"
build-requires
host-requires
dependencies
```

This is particularly relevant to packaging situations where successful construction or operation of a distribution depends on something outside the Python package dependency graph.

PEP 725 also considers cross-compilation and the distinction between the environment used to build software and the environment for which the resulting software is intended. ([peps.python.org](https://peps.python.org/pep-0725/))

## Why it matters to implementation-support research

One important class of apparent "implementation support" problems may actually be build or host requirements.

For example:

```text id="3x8q1m"
A package can require a particular environment
to BUILD successfully.
```

That is not necessarily equivalent to:

```text id="9k5v2p"
The resulting RELEASE supports only that
Python implementation at runtime.
```

These must remain separate in the research.

Conceptually:

```text id="c6w4nz"
build requirement
    ->
what is needed to construct the artifact

host requirement
    ->
what environment the resulting artifact targets

release support
    ->
what implementation the producer claims
the released distribution supports
```

The exact relationship depends on the particular build system and artifact.

## Build environment versus target environment

Cross-compilation makes the distinction especially important.

A project can have:

```text id="8j3q6v"
build environment
    !=
target/host environment
```

Therefore observing that a distribution was built using CPython does not establish that:

```text id="p5r7tc"
the resulting release supports CPython only.
```

Likewise, requiring a particular implementation or tool during the build does not automatically establish that the final runtime requires that implementation.

This is a critical anti-overclaiming rule for the research corpus.

## PEP 725 as an alternative explanation

Suppose a candidate package fails to build on PyPy.

The initial observation:

```text id="v2m8kx"
sdist cannot build on PyPy
```

does not establish:

```text id="q7c1zn"
release supports CPython only
```

The root cause might instead be:

```text id="m4x9pd"
- build dependency;
- host dependency;
- external library;
- compiler/toolchain;
- ABI requirement;
- build configuration;
- implementation-specific build backend behavior.
```

PEP 725 is therefore directly relevant when classifying these cases.

Before counting such a case as evidence for release-level implementation metadata, the research should determine whether the restriction belongs to the build/host dependency layer.

## What PEP 725 does not currently establish

PEP 725 should not be described as an existing standardized CPython/PyPy support declaration.

Its scope is external dependencies and related build/host concepts.

It does not, by itself, define:

```text id="h1v5qc"
Supported-Implementation: cpython
```

or establish a general release-level producer support policy.

Therefore the correct statement is not:

> PEP 725 cannot solve implementation requirements.

The stronger and more accurate statement is:

> PEP 725 is a relevant build/host dependency mechanism, but its semantics are not equivalent to a release-level declaration of supported Python implementations.

## Potential overlap

Some proposed implementation-support use cases may overlap with the dependency model.

For example:

```text id="x6n2wb"
CPython required to build
```

might be a build-environment constraint.

But:

```text id="r4q8vs"
CPython required to run
```

would be a runtime compatibility or support question.

And:

```text id="t3k7mp"
CPython is the only implementation
the producer promises to support
```

would be a support-policy statement.

These statements can describe the same project while representing different facts.

The research should not collapse them into one metadata concept.

## Consumer-decision test

For every source-build candidate, ask:

```text id="n8q2ya"
1. Is the observed restriction during BUILD?

2. Is it a host/target restriction?

3. Is it an external dependency?

4. Is it a Python dependency?

5. Is it an ABI/configuration requirement?

6. Is the resulting runtime actually restricted?

7. Is the producer making an explicit support-policy statement?

8. Could the existing build/host dependency model
   communicate the relevant constraint?
```

Only after these questions are answered should the case be considered as evidence for a separate implementation-support field.

## Why this is important for sdists

A major motivation being investigated is avoiding source builds that are known to fail on an unsupported implementation.

PEP 725 means that some such failures may be better understood as:

```text id="w3j6hb"
missing build/host dependency information
```

rather than:

```text id="c5r9pk"
missing implementation-support metadata
```

That distinction could substantially reduce the residual case set.

## Current conclusion

PEP 725 is a **major design dependency and alternative to investigate**.

It strengthens the research by providing a structured way to classify build, host, and external dependency restrictions before assigning them to a broader implementation-support category.

It does not currently provide an equivalent of:

```text id="d8m1qz"
Supported-Implementation
```

and therefore does not eliminate the release-support question by itself.

The correct research position is:

> **PEP 725 may absorb some apparent implementation-support cases into build/host dependency semantics. It must therefore be considered before treating source-build failures as evidence for new release-level implementation metadata.**

PEP 725 is consequently both **relevant prior art** and an **alternative explanation/mechanism**, not evidence that a new field is necessary.
