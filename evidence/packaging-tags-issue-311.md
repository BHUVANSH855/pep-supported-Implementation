# `packaging` Issue #311 — Historical Pure-Python Implementation Tags

## Purpose

This is historical prior art for implementation-specific wheel tags on
pure-Python distributions.

The discussion led to support for implementation-specific tags such as:

```text
pp3-none-any
```

## Why it matters

It demonstrates that:

```text
pure Python
```

does not necessarily mean:

```text
all Python implementations
```

An artifact can be pure Python and still have implementation-specific
requirements.

## Important correction

This issue does **not** show that wheel tags are inadequate.

In fact, it shows the opposite:

> Wheel tags are capable of representing implementation restrictions even for
> pure-Python artifacts when the artifact itself should be restricted.

The current research therefore asks a narrower question:

> What if the artifact can honestly remain `py3-none-any`, while the producer
> still declares the release unsupported on PyPy?

That is the situation demonstrated by the strongest residual cases.

## Current conclusion

Keep wheel tags as the artifact-level mechanism.

Do not use this historical issue as an argument to replace them.
