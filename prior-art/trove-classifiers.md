# Trove Classifier Prior Art

## Existing implementation classifiers

The Python packaging ecosystem already has classifiers such as:

```text
Programming Language :: Python :: Implementation :: CPython
Programming Language :: Python :: Implementation :: PyPy
```

These are useful for describing project support.

## Why they are relevant

This means a proposed standard is not inventing the concept of Python
implementation compatibility from nothing.

There is already an established vocabulary used by projects and package
indexes.

## Limitation

Trove classifiers are classification metadata.

They do not currently define the same semantics as a normative compatibility
constraint.

In particular, there is no general installer rule equivalent to:

```text
if current implementation is not listed:
    reject candidate
```

associated with these classifiers.

## Important distinction

The research should therefore say:

> Python implementation support can already be declared descriptively.

It should not say:

> Python implementation support cannot currently be declared.

The unresolved issue is whether the packaging ecosystem needs a structured,
normative, machine-actionable compatibility declaration.

## Current conclusion

Trove classifiers are important prior art and a possible source of existing
vocabulary.

Any future proposal should explain why classifiers are insufficient for the
specific use case rather than ignoring them.
