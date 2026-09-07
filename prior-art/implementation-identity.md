# Python Implementation Identity

## `sys.implementation`

PEP 421 established `sys.implementation` as the standard way for Python code
to identify interpreter implementation details.

Source:
https://peps.python.org/pep-0421/

The current dependency-specifier specification exposes implementation identity
through:

```text
implementation_name
implementation_version
platform_python_implementation
```

Source:
https://packaging.python.org/en/latest/specifications/dependency-specifiers/

## Why this matters

The research should not invent a second incompatible identity vocabulary.

A future release-support declaration should, if standardized, define exactly
how its implementation names correspond to the environment's implementation
identity.

## Potential ambiguity

The ecosystem contains alternative Python implementations and CPython-derived
runtimes.

Therefore the specification would need to answer:

```text
Does "cpython" mean exactly:

    sys.implementation.name == "cpython"

or does it mean:

    CPython-compatible runtime?
```

The first is much more deterministic.

The second is much harder to make machine-actionable.

## Current conclusion

Do not define the value vocabulary until this issue is resolved.

A field that says:

```text
Supported-Implementation: CPython
```

must have an unambiguous comparison rule.
