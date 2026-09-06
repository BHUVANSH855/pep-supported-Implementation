# packaging Issue #311

## Purpose

This document records issue #311 in the `packaging` project as historical
prior art concerning implementation-specific pure-Python wheel tags.

It should not be treated as evidence that modern wheel tags fundamentally
fail to represent Python implementation compatibility.

## Historical issue

The issue discussed generating implementation-specific `none-any` tags for
pure-Python distributions.

The eventual implementation added support for tags such as:

```text
pp3-none-any
```

This illustrates that a wheel can use the implementation component of a tag
even when the wheel contains no native extension.

## Why this is relevant

It demonstrates that implementation compatibility can exist even for a
distribution that is otherwise "pure Python".

For example, a package may intentionally restrict itself to CPython or PyPy
despite having no compiled extension.

This is relevant to the broader question of whether implementation
compatibility belongs only to artifact tags.

## What this issue does not demonstrate

It does not demonstrate that:

- wheel tags cannot express implementation compatibility;
- a new Core Metadata field is required for wheels;
- `cp3-none-any` is a missing modern standard;
- wheel tags should be replaced by Core Metadata.

The current wheel/platform compatibility specification remains the relevant
mechanism for built distributions.

## Research conclusion

The useful lesson from this issue is narrower:

> Python implementation identity can matter even for pure-Python artifacts,
> and the packaging ecosystem has historically had to account for that in
> wheel-tag generation.

The unresolved research question concerns the corresponding **release/sdist**
representation, not a failure of wheel tags.
