# Pre-PEP Thread: Requires-Implementation

**URL:** https://discuss.python.org/t/pre-pep-requires-implementation-declaring-python-implementation-compatibility-in-core-metadata/108898
**Posted:** September 6, 2026
**Category:** Packaging
**Author:** BHUVANSH855

## Thread timeline

| Post | Author | Role | Summary |
|---|---|---|---|
| 1 | BHUVANSH855 | Author | Original pre-PEP post |
| 2 | pf_moore | CPython core dev, PEP delegate | Redirect to PEP 725; questioned use cases |
| 3 | BHUVANSH855 | Author | Agreed sdist build case fits PEP 725; raised runtime case |
| 4 | eschwartz | Eli Schwartz | Challenged RestrictedPython example; pointed to PEP 425 and packaging.tags #311 |
| 5 | BHUVANSH855 | Author | Conceded wheel example; focused on sdist case |
| 6 | pf_moore | CPython core dev, PEP delegate | Confirmed no standard mechanism exists for sdist case |
| 7 | BHUVANSH855 | Author | Committed to checking PEP 725 thread |
| 8 | rgommers | NumPy/SciPy core dev, PEP 725 co-author | Challenged real-world need; raised AI concern; stale metadata objection |

## Key confirmed fact (Post 6)

Paul Moore, packaging PEP delegate, confirmed:

> "Currently, no I don't think there is [a standard way for an
> installer to know a package is CPython-only from sdist metadata
> before building]."

This is the strongest single fact in the discussion. Paul is the
packaging PEP delegate. His confirmation of the gap is on record.

## Key strategic fact (Post 6, second paragraph)

Paul also said:

> "it's only worth raising as its own individual proposal if the
> conclusion is that PEP 725 considers it out of scope (I feel like
> I'd be disappointed if that happened, though)."

If PEP 725 authors say it is out of scope, Paul's own words provide
the bridge back to a standalone PEP.

## Lessons from mistakes

1. Do not use RestrictedPython as the primary example — its wheel
   tagging is the real bug, not a metadata gap (Eli's correct objection)
2. The wheel case is fully handled by PEP 425 — do not re-open this
3. The sdist runtime/build case is the strongest remaining argument
4. Ralf's "stale metadata" objection is not yet answered
5. More concrete real-world examples are needed before proceeding
6. The AI-driven concern must be addressed directly — PyRift is the
   personal motivation, not the tool
