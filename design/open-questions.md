# Open Questions

Questions that must be answered before the PEP can be written.
In priority order. Update status as answers arrive.

---

## Q1 — Is the sdist build-failure case in scope for PEP 725?

**Priority:** Blocking
**Status:** Pending
**Where:** PEP 725 Discourse thread (post ~96)
**URL:** https://discuss.python.org/t/pep-725-specifying-external-dependencies-in-pyproject-toml-round-2/103890
**Awaiting response from:** Pradyun Gedam, Jaime Rodriguez-Guerra, Ralf Gommers

If YES — contribute to PEP 725 extension instead of standalone PEP
If NO — Paul Moore's words (Reply 2) provide the bridge back to standalone proposal

Paul Moore's exact words:
> "it's only worth raising as its own individual proposal if the
> conclusion is that PEP 725 considers it out of scope"

---

## Q2 — How do we answer the stale metadata objection?

**Priority:** Blocking
**Status:** Unresolved
**Raised by:** Ralf Gommers (September 6, 2026)

The objection: if PyPy gains abi3 wheel support or gains compatibility
with a package after `Requires-Implementation: cpython` is declared,
the metadata is wrong and harmful — it prevents PyPy from even trying.

Candidate answers to research and develop:

**Answer A — Per-release semantics**
The field is per-release, not per-project. foo 1.0 can declare
cpython-only while foo 1.1 adds pypy. Metadata does not go stale
if it is treated as a release-level declaration, not a permanent
project-level claim. Maintainers update it in new releases.

**Answer B — Requires-Python precedent**
Requires-Python has the same staleness property and the community
accepted it. A package can declare `Requires-Python: >=3.10` and
never update it even if the code later works on 3.9. The field
describes the maintainer's declared support, not technical possibility.

**Answer C — Semantic framing**
The field should be explicitly defined as "implementations the
maintainer declares as supported and tested" not "implementations
on which the package is technically impossible to run." This gives
PyPy and other implementations the ability to still try installing
an unlisted package if they choose — the field informs rather than
absolutely prevents.

**Answer D — SHOULD vs MUST semantics**
If installers SHOULD-warn rather than MUST-reject, PyPy can still
attempt installation. The field becomes informational rather than
a hard gate. This directly addresses Ralf's concern.

---

## Q3 — Are there enough real-world cases?

**Priority:** High
**Status:** Partially answered

Currently confirmed:
- guppy3 (build-time, strong evidence)

Need: 5-10 real packages where the source build itself requires a
specific implementation, a compatible wheel is unavailable, and the
incompatibility is statically knowable.

Research to do:
- Search PyPI for packages with CPython-only classifiers + sdist + no PyPy wheels
- Check their build scripts for implementation detection
- Document each case in evidence/real-world-cases.md

---

## Q4 — Should this be runtime-only or also build-time?

**Priority:** Medium
**Status:** Partially resolved

Current preference: runtime-only for v1.
Rationale: build-time case may be addressed by PEP 725 extension.
Runtime-only keeps the proposal narrow and defensible.

Risk: if PEP 725 takes the build case, the runtime case alone
may be seen as too narrow to justify a standalone PEP.

---

## Q5 — What happens when metadata conflicts with wheel tags?

**Priority:** Medium
**Status:** Unresolved

Must define authoritatively in the PEP what happens when:
- Requires-Implementation: cpython is declared
- A py3-none-any wheel is published for the same release

---

## Q6 — MUST-reject or SHOULD-warn for installers?

**Priority:** Medium
**Status:** Unresolved

Requires-Python uses MUST-reject semantics.
Is implementation compatibility equally firm?

Arguments for MUST-reject:
- Mirrors Requires-Python exactly
- Gives clear deterministic behavior

Arguments for SHOULD-warn:
- Implementation compatibility is more fluid than version compatibility
- Allows PyPy to try packages even when not listed
- Directly addresses Ralf's stale-metadata concern
- Lower barrier to adoption by maintainers

Recommendation to explore: start with SHOULD-warn in the pre-PEP,
upgrade to MUST-reject only after community feedback on semantics.
