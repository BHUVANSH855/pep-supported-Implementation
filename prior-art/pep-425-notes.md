# PEP 425 Notes — Wheel Compatibility Tags

**URL:** https://peps.python.org/pep-0425/
**Status:** Final
**Relevance:** High

## What PEP 425 does

PEP 425 defines three wheel compatibility dimensions:

```text
python tag
abi tag
platform tag
```

The Python tag identifies the implementation and Python version.

Examples include:

```text
py
cp
pp
jy
```

PEP 425 describes `py` as generic Python and says other implementations
should use `sys.implementation.name`.

## Why this matters

Wheel tags are explicitly designed so installers can determine whether a
built distribution is compatible without reading its full metadata.

That is a solved artifact-level problem.

## The residual distinction

A wheel can say:

```text
py3-none-any
```

while the producer says:

```text
CPython only
```

The strongest empirical examples in this repository are:

- RestrictedPython 8.5;
- HAX 0.3.0;
- Likepy 0.3.0;
- simple-ctx-log 0.0.3.

## Current conclusion

Do not claim wheel tags are inadequate.

Instead:

> Wheel tags solve built-artifact compatibility. The research asks whether
> release-level producer support is a separate fact that sometimes cannot be
> encoded honestly in the artifact tag.
