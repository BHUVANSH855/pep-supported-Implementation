# PEP 780 Notes

## Purpose

PEP 780 proposes ABI features as Python packaging environment markers.

It is important prior art because Python implementation identity does not
fully describe interpreter compatibility.

## Core idea

PEP 780 proposes an environment marker representing ABI characteristics of
the Python interpreter.

Examples discussed include characteristics such as:

- free-threading;
- GIL-enabled operation;
- debug builds;
- pointer width.

The proposal initially used:

```text
sys_abi_features
```

as the environment marker.

The exact proposal remains subject to discussion and revision.

## Why this matters

Consider:

```text
CPython
```

and:

```text
CPython free-threaded
```

Both have:

```text
sys.implementation.name == "cpython"
```

but a package may support one and not the other.

Therefore:

```text
implementation identity
```

cannot be treated as a complete compatibility model.

## Relationship to this research

The proposed implementation-support field should be scoped to
implementation identity.

Conceptually:

```text
Supported-Implementation
        ↓
CPython / PyPy / etc.

PEP 780-style ABI information
        ↓
free-threading / debug / pointer width / etc.

Requires-Python
        ↓
Python version

Wheel tags
        ↓
built artifact compatibility
```

These are complementary dimensions.

## Guppy3 connection

Guppy3 is particularly useful evidence because its published compatibility
information distinguishes CPython support from free-threaded CPython support.

This demonstrates why a future standard should not attempt to encode all
interpreter compatibility into one implementation-name field.

## Current conclusion

PEP 780 is not an alternative to implementation identity metadata.

It is complementary prior art and an important scope boundary.
