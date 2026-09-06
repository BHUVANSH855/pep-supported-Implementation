# Comment posted to PEP 725 thread (Post #96)

**Thread URL:** https://discuss.python.org/t/pep-725-specifying-external-dependencies-in-pyproject-toml-round-2/103890/96
**Posted:** September 6, 2026
**Author:** BHUVANSH855

---

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

---

## Current status

**No direct reply received as of September 6, 2026.**

Post #98 from Ralf Gommers (same date, 1:49pm) was a response to
Lucas Colley's question about PEP 725 progress — not to this comment.
That post confirms PEP 725 is in final stages awaiting the Packaging
Steering Council, with scope frozen.

---

## Implications

The lack of a direct reply combined with post #98 effectively confirms:

1. PEP 725 is not being extended for Python implementation identity
2. The runtime implementation compatibility case is outside PEP 725's
   current scope
3. Paul Moore's condition for a standalone proposal is met

---

## Note on the RestrictedPython example

The RestrictedPython example in this comment is used to illustrate
runtime failure (builds fine, fails at runtime on PyPy). In this
context, Eli's wheel-tagging objection does not apply because the
question is about runtime, not artifact compatibility.

However, Ralf responded to a similar point in the main thread by
noting that a downstream dependency on RestrictedPython can already
be conditioned with `platform_python_implementation` markers. This
objection does not apply to direct installation of RestrictedPython —
which is the case being raised here.

---

## Recommended next action

Wait 48-72 hours for any direct reply. If none, post a brief note
in the main Requires-Implementation thread noting that PEP 725
appears to be in final stages and the standalone route will be
explored. See `discourse/responses/ralf-gommers-pep725-post98.md`
for the full strategic analysis.