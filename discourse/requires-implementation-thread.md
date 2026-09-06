# Pre-PEP Thread: Requires-Implementation

**URL:** https://discuss.python.org/t/pre-pep-requires-implementation-declaring-python-implementation-compatibility-in-core-metadata/108898
**Posted:** September 5, 2026
**Category:** Packaging
**Stats:** 175 views, 6 likes, 2 links (as of September 6, 2026)

---

## Thread timeline

| Post | Author | Role | Key content |
|---|---|---|---|
| 1 | BHUVANSH855 | Proposer | Original pre-PEP post |
| 2 | pf_moore | CPython core dev, PEP delegate | Redirect to PEP 725; questioned use cases beyond sdist build failure |
| 3 | BHUVANSH855 | Proposer | Agreed build case fits PEP 725; raised runtime case with RestrictedPython |
| 4 | eschwartz | Eli Schwartz | Challenged RestrictedPython (PEP 425 covers it); found packaging.tags #311 |
| 5 | BHUVANSH855 | Proposer | Conceded RestrictedPython wheel example; focused on sdist case |
| 6 | pf_moore | CPython core dev | **Confirmed: no standard mechanism for sdist implementation metadata** |
| 7 | BHUVANSH855 | Proposer | Committed to checking PEP 725 thread; gracious reply |
| 8 | pf_moore | CPython core dev | **Content not yet captured — share screenshot** |
| 9 | devdanzin | Daniel Diniz | Semantic reframing: positive assertion; two concrete non-installer use cases; disclosed working relationship |

---

## Key confirmed facts

### Fact 1 — Gap is confirmed (Post 6)

Paul Moore:
> "Currently, no I don't think there is [a standard way for an installer
> to know a package is CPython-only from sdist metadata before building]."

### Fact 2 — Bridge to standalone PEP (Post 6)

Paul Moore:
> "it's only worth raising as its own individual proposal if the
> conclusion is that PEP 725 considers it out of scope"

PEP 725 post #98 shows scope is frozen. This condition is now met.

### Fact 3 — Semantic shift triggered (Post 9)

Daniel Diniz proposed positive "known to be supported" framing
over exclusionary constraint. This is the right design direction.

---

## Mistakes made and conceded

| Mistake | Who caught it | Concession |
|---|---|---|
| Used RestrictedPython as wheel example | Eli Schwartz (post 4) | Conceded. PEP 425 handles wheels. RestrictedPython has a packaging bug, not a metadata gap. |
| Used RestrictedPython in PEP 725 thread | Ralf Gommers (dependency marker counterargument) | Conceded. Downstream dependency case solved by PEP 508. Package's own declaration is still missing. |

---

## Credibility risks

| Risk | Source | Status |
|---|---|---|
| AI-driven concern | Ralf Gommers (post 8) | Addressed once. Do not repeat defense. |
| N=1 use case | Daniel Diniz (post 9) | Unresolved. Need more real-world cases or alternate implementation team endorsement. |
| Working relationship disclosure | Daniel Diniz (post 9) | Accepted. Daniel is the only public supporter so far. |

---

## Current strategic position

**Open:** PEP 725 scope is frozen (post #98). Standalone proposal route confirmed open.

**Needed before next post:** Paul's Reply 8 content. Share screenshot.