# Comment posted to PEP 725 thread

**Thread URL:** https://discuss.python.org/t/pep-725-specifying-external-dependencies-in-pyproject-toml-round-2/103890
**Posted:** September 6, 2026 (post ~96)
**Author:** BHUVANSH855

## Comment text posted

@pf_moore suggested I bring this use case here from my
Requires-Implementation thread.

I'm mainly looking at runtime compatibility, not build failures.
For example, RestrictedPython is pure Python but supports only CPython,
so it can build fine but still fail on PyPy.

From what I understand, PEP 725 currently doesn't cover this kind of
runtime implementation restriction. Is there something in the DepURL or
virtual namespace design that could handle this case, or is this outside
the current scope of PEP 725?

## Context

This comment was posted 3 months after post #95 from Lucas Colley,
who asked "what are the next steps for this effort to continue
progressing?" — the thread had been idle.

## Notes on the RestrictedPython example in this context

Eli's wheel-tagging objection (from the main thread) does not apply
here because in this context RestrictedPython is used to illustrate
runtime failure, not a build failure or wheel tagging issue.

## Awaiting response from

- Pradyun Gedam (co-author)
- Jaime Rodriguez-Guerra (co-author)
- Ralf Gommers (co-author, already engaged in main thread)

## What their answer means

| Answer | Strategic implication |
|---|---|
| "PEP 725 can handle runtime implementation via virtual deps" | Contribute to PEP 725 extension instead of standalone PEP |
| "This is outside PEP 725 scope" | Paul Moore's words bridge back to standalone proposal |
| No response within 72 hours | Post follow-up in main thread summarising findings |
