# Semantic Shift: Requires-Implementation → Supported-Implementation

**Date of shift:** September 6, 2026
**Triggered by:** Daniel Diniz reply (devdanzin, post 9)
**Reinforced by:** Ralf Gommers stale-metadata objection (post 8)

---

## Original design

```toml
[project]
requires-implementation = ["cpython", "pypy"]
```

**Semantics:** exclusionary constraint

> "This distribution is incompatible with implementations not listed.
> Installers MUST/SHOULD reject candidates on unlisted implementations."

Mirrors `Requires-Python` exactly.

---

## The problem with the original design

Ralf Gommers raised this objection:

> "Packages that declare 'doesn't support PyPy' would then be unhelpfully
> out of date. Concrete example: there's some effort ongoing to make PyPy
> understand abi3 wheels. Packages that declare 'doesn't support PyPy'
> would then be unhelpfully out of date."

An exclusionary field freezes a compatibility claim that should be
dynamic. PyPy, GraalPy, and other implementations actively improve
compatibility. A static hard block makes their jobs harder.

This is the **stale metadata problem** — the strongest technical
objection the thread has produced.

---

## Daniel Diniz's reframing (post 9)

> "I think the better objective is to list what is known to be supported,
> and that should usually be evergreen. IMHO the positive assertion helps
> when picking the extension stack to build your product from."

Instead of:

```
Requires-Implementation: cpython   ← "only cpython allowed"
```

Consider:

```
Supported-Implementation: cpython  ← "cpython is known to work"
```

---

## Why the positive assertion is better

### 1. Staleness risk is reduced

`Supported-Implementation: cpython` means the maintainer has tested
and confirmed CPython support for this release. It does NOT mean PyPy
is permanently impossible. When PyPy gains compatibility, new releases
add `Supported-Implementation: pypy`. Old releases remain accurate —
they never claimed PyPy was impossible.

### 2. Ralf's objection is directly addressed

A positive-only assertion does not prevent PyPy from attempting
installation. It informs tooling about what is confirmed, not what
is forbidden.

### 3. Daniel's non-installer use cases are served

Fuzzing and large-scale compatibility testing need to know what is
known to be supported, not what is explicitly excluded.

### 4. Ecosystem signal value is preserved

A machine-readable picture of implementation diversity across PyPI
is useful beyond installation tooling.

---

## The critical open question: absence semantics

If a distribution declares `Supported-Implementation: cpython`
and `pypy` is absent, what should a consumer infer?

| Option | Meaning | Installer behavior |
|---|---|---|
| A — Exclusionary | PyPy is unsupported | MUST reject |
| B — Unknown | PyPy status is unknown | Allow, possibly warn |
| C — Advisory | CPython confirmed; others may work | Allow, no warning |

**The entire design depends on which interpretation is chosen.**

- For Daniel's tooling: B and C both work
- For Ralf's staleness concern: B or C resolve it; A does not
- For installer candidate selection: A is most powerful but risky
- For PyPy's ability to gain compatibility over time: B or C

**Current recommendation:** Interpretation B — absence means no claim
has been made, not that the implementation is incompatible. This makes
the field informational with SHOULD-warn installer semantics rather
than a hard gate.

---

## Comparison table

| Property | Requires-Implementation | Supported-Implementation |
|---|---|---|
| Semantics | Hard exclusion | Positive declaration |
| Absence means | Unrestricted (ambiguous) | No claim made |
| Stale metadata risk | High | Low |
| Installer behavior | MUST reject | SHOULD warn |
| Ralf's objection | Not addressed | Addressed |
| Daniel's use case | Partially served | Fully served |
| Requires-Python analogy | Close | Less close |
| PEP acceptance risk | Higher | Lower |

---

## Current design direction

Use `Supported-Implementation` with:

- Positive-only assertion semantics
- Open value set from `sys.implementation.name` (PEP 421)
- Absence = no claim, not incompatibility
- Installer behavior: SHOULD-warn if running implementation not listed
- Per-release field, not per-project permanent claim

**Field name candidates for community discussion:**
- `Supported-Implementation` (clearest)
- `Known-Implementations` (most neutral)
- `Tested-Implementation` (most precise)

Name should follow from the absence-semantics decision, not precede it.