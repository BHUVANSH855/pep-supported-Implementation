# Design decisions

Current thinking on each design dimension. All of these are open
for discussion — this reflects the current state of the research,
not a final specification.

---

| Dimension | Current direction | Notes |
|---|---|---|
| Field name | `Supported-Implementation` | Open to alternatives |
| TOML key | `supported-implementation` | Follows field name |
| Value type | List of strings | e.g. `["cpython", "pypy"]` |
| Value vocabulary | `sys.implementation.name` per PEP 421 | Open set, no registry needed |
| Semantics | Positive declaration of known support | Not an exclusionary constraint |
| Missing field | No claim made | Does not mean incompatible |
| Multiple values | Each listed implementation is confirmed supported | OR semantics |
| Version constraints | Out of scope for initial proposal | Can be added later |
| Scope | Runtime compatibility only | Build-time is separate |
| Installer behaviour | SHOULD-warn if implementation not listed | Not MUST-reject |
| `Supported-Platform` | Leave unchanged for now | Different scope |
| Backwards compatibility | Field is optional | No existing packages affected |

---

## Why positive semantics rather than exclusionary

The original framing was `Requires-Implementation: cpython`, which
would mean "only CPython is allowed." This creates a staleness problem:
if PyPy later gains compatibility with a package, the old metadata
becomes a harmful hard block that cannot be retroactively corrected
for published releases.

A positive declaration (`Supported-Implementation: cpython`) means
"CPython is known to work." It does not mean PyPy cannot work. As
alternate implementations improve, new releases can simply add them
to the list. Old releases remain accurate — they never claimed
incompatibility, only declared what was confirmed.

---

## Why not just use existing mechanisms

| Mechanism | What it does | What it does not do |
|---|---|---|
| Wheel tags (PEP 425) | Artifact-level implementation filtering | Does not apply to sdists |
| `implementation_name` markers (PEP 508) | Conditional dependency selection | Does not describe the package's own compatibility |
| Trove classifiers | Descriptive declaration | Not a normative installer constraint |
| `Requires-Python` | Python version constraint | Does not identify implementation |
| `Supported-Platform` | OS/CPU for binary dists (semantics undefined) | Wrong scope and name |