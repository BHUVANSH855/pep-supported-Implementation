# pep-supported-implementation

Research repository tracking the investigation into a potential
`Supported-Implementation` Core Metadata field for Python packaging.

**This is not a PEP draft.** It is a structured evidence base for a
pre-PEP Discourse discussion started on September 6, 2026.

---

## Status

Active research. Community discussion ongoing. No PEP number assigned.

**Design has evolved:** The original proposal was `Requires-Implementation`
(hard exclusionary constraint). After community feedback, the better
framing is `Supported-Implementation` (positive declaration of known
support). See `design/semantic-shift.md`.

---

## The question

Python packaging already standardises:

- Implementation identity at runtime — `sys.implementation.name` (PEP 421)
- Implementation-compatible wheel selection — wheel filename tags (PEP 425)
- Implementation-specific dependencies — `implementation_name` markers (PEP 508)
- Python version constraints — `Requires-Python` in Core Metadata

What does not exist is a distribution-level Core Metadata field
expressing which Python implementations a project is known to support.

Paul Moore (CPython core developer, packaging PEP delegate) confirmed
on September 6, 2026:

> "Currently, no I don't think there is [a standard way for an installer
> to know a package is CPython-only from sdist metadata before building]."

---

## Primary Discourse thread

https://discuss.python.org/t/pre-pep-requires-implementation-declaring-python-implementation-compatibility-in-core-metadata/108898

## PEP 725 thread comment (post #96, awaiting response)

https://discuss.python.org/t/pep-725-specifying-external-dependencies-in-pyproject-toml-round-2/103890/96

---

## Repository structure

```
prior-art/          — analysis of existing PEPs and specifications
discourse/          — thread posts and all community responses
evidence/           — real-world package cases and tooling issues
design/             — design decisions, open questions, semantic shift
```

---

## Key findings

**Gap confirmed:** Paul Moore confirmed no standard mechanism exists for
an sdist to declare implementation compatibility before a build attempt.

**Strongest evidence:** guppy3 — its `setup.py` explicitly detects
non-CPython, warns compilation failure is expected, yet publishes an
sdist with no pre-build metadata signal.

**Semantic shift:** Daniel Diniz's reply proposed a positive
`Supported-Implementation` declaration rather than an exclusionary
`Requires-Implementation` constraint. This directly addresses Ralf
Gommers's stale-metadata objection and broadens the value proposition
to non-installer tooling (fuzzing, large-scale compatibility testing).

**PEP 725 status:** Post #98 from Ralf (September 6, 2026) confirms
PEP 725 is ready and awaiting the Packaging Steering Council — it is
not being extended for Python implementation identity. Your comment
(post #96) has received no direct reply yet.

**Standalone proposal route:** Paul Moore explicitly stated this is
"only worth raising as its own individual proposal if PEP 725 considers
it out of scope." Ralf's post #98 confirms PEP 725 is effectively closed
to new scope additions at this stage.

---

## What this repository does NOT claim

- That `Supported-Implementation` is proven necessary
- That PEP 725 cannot solve the build-time case in theory
- That every CPython-only package needs new metadata
- That wheel tags are insufficient for built distributions
- That RestrictedPython is a good motivating example (it is not)

---

## Author

Bhuvansh (BHUVANSH855 on discuss.python.org)
CPython contributor, Python docs translation coordinator (Punjabi),
builder of PyRift (CPython/PyPy comparison toolkit)
