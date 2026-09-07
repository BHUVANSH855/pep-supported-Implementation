# Requirement vs Supported Implementation

The original proposal used the name `Requires-Implementation`.

The empirical cases now make the semantic choice more important.

## Requirement

```text
Requires-Implementation: cpython
```

Possible meaning:

> This release cannot be used with implementations other than CPython.

This is analogous to:

```text
Requires-Python: >=3.10
```

### Problem

A technical impossibility claim is stronger than a support-policy claim.

A project may say:

```text
CPython only
```

because it has only tested CPython.

That does not prove PyPy is technically incapable of running it.

---

## Support declaration

```text
Supported-Implementation: cpython
```

Possible meaning:

> The producer explicitly supports this release on CPython.

This better matches the strongest evidence cases.

It also avoids turning every omitted implementation into a claim of technical
incompatibility.

### But normative semantics are still required

A resolver needs to know:

```text
Supported-Implementation: cpython
```

means what?

Possible definitions:

### Definition A — exhaustive

Only listed implementations are supported.

Then:

```text
PyPy
```

is a rejection.

### Definition B — positive guarantee

The listed implementations are explicitly supported, but unlisted
implementations remain unknown.

Then:

```text
PyPy
```

is not automatically rejected.

Definition B is safer but provides less candidate filtering.

---

## What the empirical cases suggest

For RestrictedPython, HAX, Likepy and similar releases, the producer is making
a stronger statement than “we tested CPython”.

The documentation explicitly says other implementations are unsupported.

Those cases could justify exhaustive semantics.

But the standard should not assume every implementation classifier or support
declaration is equivalent to that strong claim.

---

## Recommended research direction

Do not finalize the field name yet.

First determine:

1. whether consumers need rejection semantics;
2. whether support declarations are intended to be exhaustive;
3. whether omission means unknown;
4. how implementation identity is matched;
5. how forks are handled;
6. how support declarations interact with ABI features;
7. whether a future classifier could provide the same semantics.

Only then should a field name be frozen.
