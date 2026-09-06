# Design Decision Matrix

Last updated: September 6, 2026 (post Daniel Diniz reply, post Ralf PEP 725 post #98)

---

## Core design decisions

| Decision | Previous | Current | Confidence | Changed? |
|---|---|---|---|---|
| Field name | `Requires-Implementation` | `Supported-Implementation` (TBD) | Low | YES |
| Semantics | Hard exclusionary | Positive declaration of known support | Medium | YES |
| TOML key | `requires-implementation` | `supported-implementation` (TBD) | Low | YES |
| Value type | List of strings | List of strings | Medium | No |
| Value vocabulary | `sys.implementation.name` per PEP 421 | Same | High | No |
| Missing field means | No restriction | No claim made | High | Clarified |
| Multiple values | OR semantics | Each listed impl is confirmed supported | High | Clarified |
| Version constraints | Out of scope for v1 | Out of scope for v1 | Medium | No |
| `Supported-Platform` | Not repurposed | Not repurposed | High | No |
| Installer behavior | MUST reject | SHOULD warn | Medium | YES |
| PyPI behavior | Expose via data-core-metadata | Same | Medium | No |
| Classifiers | Remain descriptive | Same | High | No |
| Backwards compat | Field optional, absent = unrestricted | Field optional, absent = no claim | High | Clarified |

---

## PEP 725 scope — now effectively resolved

**Post #98 from Ralf Gommers (September 6, 2026):**

> "This PEP is ready; PEP 804 is quite close too, and will get a
> (hopefully last) update soon, when the Packaging Steering Council
> is seated."

PEP 725 is in final stages awaiting the Packaging Steering Council.
It is not being extended for Python implementation identity. Your
comment (post #96) has not received a direct reply, but post #98
makes clear the PEP scope is frozen.

**Implication:** Paul Moore's condition ("only worth raising as its own
individual proposal if PEP 725 considers it out of scope") is now
effectively met. PEP 725's scope is closed. A standalone proposal
is warranted.

---

## Unresolved questions (priority order)

### 1. Absence semantics — CRITICAL / BLOCKING

If `Supported-Implementation: cpython` is declared and `pypy` is absent:

| Option | Meaning | Verdict |
|---|---|---|
| A — Exclusionary | PyPy unsupported, MUST reject | Creates staleness — avoid |
| B — Unknown | PyPy status unknown, allow + warn | Safe, honest, recommended |
| C — Advisory | CPython confirmed, others may work, allow | Weakest signal |

**Current preference: B.** Absence means no claim, not incompatibility.

### 2. Field name — needs community input

Wait for the semantics decision before committing to a name.
Do not use `Requires-Implementation` going forward in public posts.

### 3. Wheel / metadata conflict

What happens if `Supported-Implementation: cpython` is declared but
a `py3-none-any` wheel is published?

Preferred framing: the field operates at project/release level; wheel
tags operate at artifact level. These answer different questions.
Not a conflict by design.

### 4. SHOULD-warn vs MUST-reject

Current preference: SHOULD-warn for initial version.
Can be strengthened in a future revision if adoption demonstrates the
positive-only semantics work well in practice.

### 5. How to answer the "N=1" problem

Daniel disclosed he works with the proposer and said the N of people
with this need is 1 or very near to it. This is the biggest credibility
problem in the thread.

**Before re-engaging:** find at least one more independent person or
tool that would benefit. PyRift users, GraalPy team, or PyPy team
members are good candidates to reach.