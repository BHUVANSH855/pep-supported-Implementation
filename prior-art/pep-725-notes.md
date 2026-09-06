# PEP 725 Notes

## Purpose

PEP 725 is important prior art because it addresses dependency information
associated with building and hosting software.

It must be considered before proposing a new field for build-time Python
implementation requirements.

## What PEP 725 addresses

PEP 725 provides a framework for describing external dependencies and
distinguishes requirements relevant to different stages of the packaging
process.

This includes concepts related to:

- build requirements;
- host requirements;
- runtime external dependencies.

## Relevance to Python implementations

The question for this research is whether a Python implementation itself
could be modeled using the PEP 725 dependency framework.

For example, conceptually:

```text
build requires CPython
```

could be treated as a dependency on a virtual implementation.

However, the current PEP 725 specification does not define CPython, PyPy,
or other Python implementations as a standardized virtual dependency
vocabulary for this purpose.

## Important distinction

There are two different statements:

```text
The released package supports CPython.
```

and:

```text
The package must be built using CPython.
```

The first is a release/runtime compatibility statement.

The second is a build-environment requirement.

A future design should avoid assuming that these are the same metadata field.

## Relationship to this research

PEP 725 provides a possible alternative direction for the build-time case.

Therefore the existence of PEP 725 means that a new
`Requires-Implementation` field should not be proposed without first
explaining why build-time implementation requirements cannot or should not be
represented there.

At the same time, the current PEP 725 specification does not itself provide
the release-level runtime implementation compatibility declaration being
investigated here.

## Current conclusion

PEP 725 is:

- relevant to build-time implementation requirements;
- not currently a direct solution for release-level implementation support;
- a potential alternative or complementary mechanism;
- an important unresolved design dependency.

The research should therefore avoid claiming that PEP 725 is irrelevant or
that it cannot be extended.
