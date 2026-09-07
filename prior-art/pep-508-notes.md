# PEP 508 / Dependency Specifier Notes

**URL:** https://packaging.python.org/en/latest/specifications/dependency-specifiers/
**Relevance:** High

## Existing implementation environment markers

Dependency markers include:

```text
platform_python_implementation
implementation_name
implementation_version
```

For example:

```text
Requires-Dist: cffi; implementation_name == "pypy"
```

## What markers solve

They answer:

> Under which environments is this dependency required?

This is valuable and already standardized.

## What they do not directly solve

They do not express:

> This distribution itself is unsupported on PyPy.

There is no normal `Requires-Dist` entry whose semantic meaning is:

```text
reject the current distribution when its own marker is false
```

## Important warning

This does not mean PEP 508 is irrelevant.

A package may solve an implementation-specific dependency problem completely
using markers.

Therefore the research corpus must distinguish:

```text
implementation-specific dependency
```

from:

```text
implementation-specific release support
```

## Current conclusion

PEP 508 is a complementary mechanism.

The remaining question is whether release-level support needs a separate
normative representation.
