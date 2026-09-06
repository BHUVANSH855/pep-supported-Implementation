# Design Decision Matrix

Decisions that must be made before writing the PEP.
Each row shows the question, the current preferred answer, and
confidence level. Low confidence = needs community input first.

| Decision | Preferred answer | Confidence | Notes |
|---|---|---|---|
| Field name | `Requires-Implementation` | High | Mirrors Requires-Python |
| TOML key | `requires-implementation` | High | Mirrors requires-python |
| Value type | List of strings | Medium | Could be specifier syntax — open question |
| Value vocabulary | sys.implementation.name per PEP 421 | High | Open set, no registry needed |
| Scope | Runtime only, not build | Medium | Build case may go to PEP 725 |
| Missing field means | No restriction declared | High | Must NEVER default to cpython |
| Multiple values | OR semantics | High | Either listed impl is acceptable |
| Version constraints | Out of scope for v1 | Medium | Defer to future extension |
| Supported-Platform | Not repurposed | High | Different scope, poor name |
| Wheel interaction | Must define py3-none-any conflict rule | Low | Open question — unresolved |
| Installer behavior | MUST reject or SHOULD warn? | Low | Needs community input |
| PyPI behavior | Expose via data-core-metadata | Medium | No upload enforcement initially |
| Classifiers | Remain descriptive, this field normative | High | Clearly stated distinction |
| Environment markers | Remain dependency-level | High | Different semantic layer |
| Backwards compat | Field optional, absent = unrestricted | High | No existing packages break |

## Unresolved questions — must answer before proceeding

### 1. Stale metadata (Ralf Gommers objection — CRITICAL / BLOCKING)

If a package declares `Requires-Implementation: cpython` and PyPy later
gains compatibility, the declaration becomes wrong and potentially harmful.

Candidate answers to develop:

A) The field is per-release, not per-project.
   foo 1.0 can be cpython-only while foo 1.1 adds pypy.
   Metadata does not go stale because each release declares independently.

B) Requires-Python has the same staleness property and is accepted.
   Python 3.8 support can be re-added but old declarations remain.
   The community accepted this tradeoff.

C) The field should describe "tested and supported" not
   "technically impossible forever." Installers can treat it as advisory
   for implementations not listed, not as a hard block.

Status: UNRESOLVED. Must resolve before re-engaging in thread.

### 2. Wheel / metadata conflict

What happens if a package declares:
```
Requires-Implementation: cpython
```
but publishes a `py3-none-any` wheel?

Options:
- Wheel tags are authoritative, metadata is advisory
- Metadata is authoritative, wheel tag is a lower bound
- Publishing a contradictory combination is a validation error

Status: UNRESOLVED. Must define in PEP.

### 3. Is build-failure or runtime-failure the primary use case?

guppy3 is a build-failure case.
A pure-Python package like objgraph is a runtime-failure case.

These have different relationships to PEP 725:
- Build case: Paul Moore thinks PEP 725 is the right venue
- Runtime case: Ralf Gommers says PEP 725 does not cover it

The proposal needs to clearly state which case it is solving,
or explicitly solve both and justify why they belong together.

Status: PARTIALLY RESOLVED. Current preference is runtime-only for v1.

### 4. PEP 725 scope question — BLOCKING

Is the sdist build-failure case in or out of scope for PEP 725?
Awaiting response from PEP 725 authors in Discourse thread.

URL: https://discuss.python.org/t/pep-725-specifying-external-dependencies-in-pyproject-toml-round-2/103890

Status: PENDING. Do not proceed until answered.
