# `Requires-Python` Prior Art

**Current specification:** Core Metadata 2.6
**Source:** https://packaging.python.org/en/latest/specifications/core-metadata/

`Requires-Python` specifies the Python versions with which a distribution is
compatible.

Example:

```text
Requires-Python: >=3.10
```

Installation tools may use this field when selecting project versions.

## What it solves

```text
Python language/version compatibility
```

## What it does not solve

It cannot express:

```text
CPython only
```

or:

```text
CPython + PyPy
```

The dimensions are independent:

```text
Python version
    Requires-Python

Python implementation
    separate question
```

A package can support CPython 3.10+ but not PyPy 3.10.

Current conclusion:

> `Requires-Python` is an important existing analogue, but it does not
> provide implementation identity/support semantics.
