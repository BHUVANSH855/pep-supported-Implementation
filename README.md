# pep-requires-implementation

Research repository tracking the investigation into a potential
`Requires-Implementation` Core Metadata field for Python packaging.

**This is not a PEP draft.** It is a structured evidence base for a
pre-PEP Discourse discussion started on September 6, 2026.

## Status

Active research. Community discussion ongoing. No PEP number assigned.

## The question

Python packaging already standardises:

- Implementation identity at runtime — `sys.implementation.name` (PEP 421)
- Implementation-compatible wheel selection — wheel filename tags (PEP 425)
- Implementation-specific dependencies — `implementation_name` markers (PEP 508)
- Python version constraints — `Requires-Python` in Core Metadata

What does not exist is a distribution-level Core Metadata field
expressing that a project's runtime is compatible only with certain
Python implementations.

Paul Moore (CPython core developer, packaging PEP delegate) confirmed
on September 6, 2026:

> "Currently, no I don't think there is [a standard way for an installer
> to know a package is CPython-only from sdist metadata before building]."

## Primary Discourse thread

https://discuss.python.org/t/pre-pep-requires-implementation-declaring-python-implementation-compatibility-in-core-metadata/108898

## Repository structure

```
prior-art/          — analysis of existing PEPs and specifications
discourse/          — thread posts and community responses
evidence/           — real-world package cases and tooling issues
design/             — design decisions and open questions
```

## Key finding so far

The strongest concrete case is **guppy3**, a package whose `setup.py`
explicitly detects non-CPython and states compilation failure is
expected — yet publishes an sdist with no pre-build metadata signal
of this incompatibility.

## What this repository does NOT claim

- That `Requires-Implementation` is proven necessary
- That PEP 725 cannot solve this
- That every CPython-only package needs new metadata
- That wheel tags are insufficient for built distributions

## Author

Bhuvansh (BHUVANSH855 on discuss.python.org)
CPython contributor, Python docs translation coordinator (Punjabi),
builder of PyRift (CPython/PyPy comparison toolkit)
