# Open Questions

Last updated: September 6, 2026

---

## Q1 — Absence semantics

**Priority:** Blocking
**Status:** Unresolved

If `Supported-Implementation: cpython` is declared and `pypy` absent:

- A: PyPy unsupported → MUST reject
- B: PyPy unknown → allow, SHOULD warn
- C: CPython confirmed, others may work → allow, no warning

**Current preference: B.**
Must resolve before any PEP draft is written.

---

## Q2 — Is PEP 725 scope closed to this use case?

**Priority:** High
**Status:** Effectively resolved

Ralf Gommers, PEP 725 co-author, posted (post #98, September 6, 2026):

> "This PEP is ready; PEP 804 is quite close too, and will get a
> (hopefully last) update soon, when the Packaging Steering Council
> is seated."

The PEP is in final stages, scope is frozen. Your comment at post #96
has not received a direct reply. No new scope is being added.

**Conclusion:** Paul Moore's condition ("only worth raising as its own
individual proposal if PEP 725 considers it out of scope") is now met.
The standalone proposal route is open.

**One action remaining:** Post a brief follow-up in your main thread
noting that PEP 725 appears to be in final stages and unlikely to
absorb this use case, and that you will proceed with exploring a
standalone proposal.

---

## Q3 — More real-world cases needed

**Priority:** High
**Status:** One strong case confirmed (guppy3)

Ralf and Paul both said the use case N is too small. Daniel said N=1.
This is the biggest credibility problem.

**Research needed:** Find 3-5 more packages like guppy3:
- CPython classifier declared
- sdist published
- No PyPy wheel
- Build script detects non-CPython

Good search: PyPI packages with `Programming Language :: Python ::
Implementation :: CPython` AND published sdist AND no `pp*` wheel.

Also consider: reach out to PyPy team or GraalPy team to ask whether
they would use such a field. An endorsement from an alternate
implementation team is far stronger than a second individual.

---

## Q4 — Field name

**Priority:** Medium
**Status:** Unresolved, needs community input after semantics settled

Candidates:
- `Supported-Implementation` — clearest, positive framing
- `Known-Implementations` — neutral
- `Tested-Implementation` — most honest

Do not commit until Q1 (absence semantics) is resolved.

---

## Q5 — Addressing the "N=1" credibility problem

**Priority:** Medium
**Status:** Unresolved

Daniel said the N of people with this need is 1 or very near to it,
then disclosed he works with the proposer.

Options:
1. Find more real-world packages (Q3 above)
2. Reach out to alternate implementation maintainers
   (PyPy team, GraalPy team, RustPython, MicroPython)
3. Find tooling that already tries to solve this with workarounds
   (pip-audit, pip-tools, conda, uv) and document the workaround
4. Survey PyPI: count packages declaring CPython-only classifiers
   that have an sdist but no PyPy wheel — the ecosystem scale matters

---

## Q6 — AI concern response

**Priority:** Medium (credibility)
**Status:** Addressed once, monitor for recurrence

Facts on record:
- AI tools used for prior art research
- PyRift is a real toolkit for CPython/PyPy comparison
- Daniel provided independent use cases (then disclosed relationship)
- Research repo is public

Do not over-defend. Answer once clearly, then move on.
Repeated defensiveness looks worse than the original concern.

---

## Q7 — What to post next in the Discourse thread

**Priority:** High
**Status:** Draft ready in discourse/next-reply-draft.md

Do not post until Paul's Reply 8 content is read.
Share screenshot of Reply 8 immediately.