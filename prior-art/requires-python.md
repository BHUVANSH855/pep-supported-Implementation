# Requires-Python Prior Art

## Purpose

`Requires-Python` is the existing Core Metadata mechanism for declaring the
Python versions compatible with a distribution.

## Example

A project can declare:

```toml
requires-python = ">=3.10"
```

which maps to:

```text
Requires-Python: >=3.10
```

## What it solves

It answers:

> Which Python versions does this distribution support?

This is already a standardized and installer-relevant compatibility
dimension.

## What it does not solve

It does not distinguish implementations.

For example:

```text
Requires-Python: >=3.10
```

does not express:

```text
CPython only
```

or:

```text
CPython and PyPy
```

## Relationship to implementation compatibility

The two dimensions are independent:

```text
Python version
    Requires-Python

Python implementation
    implementation compatibility
```

A package could theoretically support:

```text
CPython 3.10+
```

while not supporting:

```text
PyPy 3.10
```

despite both satisfying the same Python version constraint.

## Current conclusion

`Requires-Python` solves Python-version compatibility but leaves Python
implementation identity as a separate dimension.
