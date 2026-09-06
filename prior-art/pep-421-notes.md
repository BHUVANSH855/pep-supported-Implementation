# PEP 421 — sys.implementation

**URL:** https://peps.python.org/pep-0421/
**Status:** Final
**Relevance:** High — provides the vocabulary for implementation identity

## What it does

Introduces `sys.implementation` as a standard namespace for Python
implementation information. Defines `sys.implementation.name` as a
lowercase string identifying the running implementation.

## Known values

| Implementation | sys.implementation.name |
|---|---|
| CPython | `cpython` |
| PyPy | `pypy` |
| Jython | `jython` |
| IronPython | `ironpython` |
| GraalPy | `graalpy` |
| MicroPython | `micropython` |

**Important:** PEP 421 does NOT define a closed registry.
The set is open. Future implementations use their own name.

## Relevance to this proposal

If `Requires-Implementation` is defined, values should correspond
to `sys.implementation.name` per PEP 421. No new vocabulary needed.
No central registry required.

## What PEP 421 does NOT do

- It does not define packaging metadata
- It does not define a list of allowed implementation names
- It does not create any installer behavior

## Key quote

> "The name must be a valid Python identifier and should not be
> changed by the implementation after initialization."
