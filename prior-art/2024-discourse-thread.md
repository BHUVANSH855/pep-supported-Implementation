# Prior Discussion — Python Implementation in Metadata (January 2024)

**URL:** https://discuss.python.org/t/python-implementation-in-metadata/42653

## What was asked

The original thread asked why Python implementation information was not stored
in package metadata and described a desire to force PyPy for an application.

## Key responses

Paul Moore questioned the use case for a library and suggested classifiers
as the appropriate way to declare which implementations a project is willing
to support without forcing anything.

Sinoroc suggested Trove classifiers.

C.A.M. Gerlach pointed to classifiers, wheel tags, and backend/plugin logic as
existing mechanisms.

## Why this matters

The thread establishes that:

```text
implementation support can already be declared descriptively
```

It does not establish that:

```text
implementation support cannot have normative resolver semantics
```

The current research therefore treats the thread as **counter-evidence that
must be answered**, not as a rejection of the entire concept.

## Correct way to describe the difference

Original use case:

```text
user wants to force a development/application interpreter
```

Current research:

```text
release producer wants to declare implementation support
for candidate selection, especially where artifacts are generic
```

Those are related but distinct problems.

## Research conclusion

The 2024 discussion strengthens the requirement that any new proposal must
explain why classifiers are insufficient for the new semantics.
