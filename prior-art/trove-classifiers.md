# Trove Classifier Prior Art

## Existing vocabulary

Python packages can publish:

```text
Programming Language :: Python :: Implementation :: CPython
Programming Language :: Python :: Implementation :: PyPy
```

This is important evidence because the ecosystem already has a concept of
implementation support.

## What classifiers provide

They are useful for:

- PyPI display;
- search/filtering;
- human communication;
- ecosystem classification.

## What is not currently defined

There is no general Core Metadata rule saying:

```text
if current implementation is not represented by an implementation classifier:
    reject this distribution candidate
```

That is the key semantic distinction.

## 2024 discussion

Paul Moore argued that classifiers were appropriate for the original use case
of declaring which implementations a project is willing to support without
forcing anything.

Source:
https://discuss.python.org/t/python-implementation-in-metadata/42653

This should be treated as an important counterargument to a new field.

## Current research question

The proposal should therefore answer:

> Why does the ecosystem need a second, normative representation rather than
> continuing to use classifiers?

If that question cannot be answered convincingly, the new field should not be
standardized.
