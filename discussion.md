# Discussion Record

This file records the main arguments and observations from the
`Requires-Implementation` discussion.

## Paul Moore: identify the concrete use cases

Paul Moore asked for more concrete examples before treating a new metadata
field as necessary.

A particularly relevant observation was whether there is currently a
standard way for an installer to determine, from sdist metadata before
building, that a package is CPython-only.

The research recorded here treats that as an important question rather than
assuming the answer in advance.

## PEP 725

PEP 725 was suggested as an area to investigate before introducing a new
metadata mechanism.

PEP 725 is concerned with external dependencies and distinguishes requirements
needed at different stages of the packaging process.

The current PEP does not define Python implementations such as CPython or
PyPy as a standard virtual dependency vocabulary.

That means PEP 725 is relevant prior art, but the current specification does
not directly provide a release-level Python implementation compatibility
field.

This leaves an important design question:

> Should build-time implementation requirements be handled as part of the
> PEP 725 dependency model, rather than by a new implementation compatibility
> metadata field?

The research does not currently resolve that question.

## Ralf Gommers: staleness and positive declarations

A concern raised during the discussion is that an explicit negative
compatibility declaration can become stale.

For example:

```text
Requires-Implementation: cpython
```

would imply that other implementations are not acceptable.

If PyPy later becomes compatible, an old release could still contain a
negative statement that is no longer technically true.

This motivated investigation of a positive declaration:

```text
Supported-Implementation: cpython
```

Such a declaration can be interpreted as the set of implementations the
project explicitly supports for that release, rather than as an exhaustive
list of implementations that are technically capable of running it.

That distinction is important and remains an open semantic question.

## Daniel Diniz: non-installer tooling

Daniel Diniz identified use cases beyond ordinary installation.

Large-scale compatibility tooling may need to classify thousands of projects
against multiple Python implementations.

Examples include:

- compatibility testing across CPython, PyPy, and other implementations;
- fuzzing projects against multiple interpreters;
- selecting packages that are worth attempting on a particular interpreter;
- avoiding expensive builds where compatibility can already be determined
  from metadata.

This suggests that implementation compatibility metadata could have value
even when the final decision is not made by an installer.

## RestrictedPython

RestrictedPython was discussed as a possible CPython-only example.

The research does not currently treat it as a primary proof case.

The stronger evidence comes from projects whose build or runtime behavior
explicitly checks the Python implementation.

## Guppy3

`guppy3` is currently the strongest concrete example in this repository.

Its build configuration explicitly checks:

```python
sys.implementation.name != "cpython"
```

and its published project information states that PyPy and other
implementations are unsupported.

This is a useful example because the implementation restriction exists at the
project/release level while an sdist remains a source artifact that must
normally be built into a wheel.

It does not prove that a new metadata field is required. It demonstrates the
kind of information the proposed field would attempt to represent.

## ABI compatibility

The Guppy3 case also exposes a second dimension.

Its published compatibility information distinguishes ordinary CPython
compatibility from free-threaded CPython compatibility.

That means:

```text
implementation = cpython
```

does not necessarily mean:

```text
all CPython ABI configurations = supported
```

PEP 780 is therefore relevant prior art for ABI-level compatibility.

A proposed implementation field should not attempt to replace ABI feature
metadata.

## Sdist build avoidance

There are existing packaging discussions about avoiding unwanted attempts to
build sdists.

Those discussions are relevant because they establish a broader ecosystem
problem:

> deciding whether an sdist should be built can itself be useful information
> before performing an expensive or potentially failing build.

However, these discussions do not establish that implementation metadata is
the required solution.

They should therefore be treated as problem-space evidence rather than proof
of a particular design.

## Current interpretation

The discussion supports the following research position:

1. There are real implementation-specific compatibility cases.
2. Existing wheel tags solve the problem for already-built wheels.
3. PEP 508 solves conditional dependency selection.
4. Trove classifiers provide descriptive implementation information.
5. Static sdist metadata makes additional release metadata technically
   plausible.
6. There remains no dedicated normative Core Metadata field for release-level
   Python implementation compatibility.
7. It is still unresolved whether that gap should be solved by a new field,
   an extension to another packaging mechanism, or improved tooling.

That is the current boundary of the evidence.
