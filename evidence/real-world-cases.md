# Real-World Package Cases

Last updated: September 6, 2026

---

## TIER 1 — Strong evidence (use in discussions)

### guppy3

**PyPI:** https://pypi.org/project/guppy3/
**Version:** 3.1.7 (May 11, 2026)
**Classifier:** `Programming Language :: Python :: Implementation :: CPython`

**PyPI declaration:**
> "This package is CPython only; PyPy and other Python implementations
> are not supported."

**Artifacts:**
- `guppy3-3.1.7.tar.gz` (sdist — no implementation tag)
- `guppy3-3.1.7-cp314-cp314-*.whl` (CPython-specific wheels)
- No PyPy wheels

**Build script evidence (setup.py):**

```python
if sys.implementation.name != 'cpython':
    print(
        'setup.py: Warning: This guppy package only supports CPython.',
        'Compilation failure expected, but continuing anyways...'
    )
```

Then compiles C extensions: `guppy.sets.setsc` and `guppy.heapy.heapyc`.

**Why this is strong:**
Build system detects non-CPython and explicitly expects failure. An sdist
is published with no pre-build metadata signal. No PyPy wheel exists, so
a PyPy user receives the sdist and encounters a build failure with no
normative warning from the installer.

**NOT verified:** We have not run `pypy -m pip install guppy3`.
Cite source code and PyPI declaration only.

---

### Daniel Diniz's tooling (non-installer use cases)

**Source:** devdanzin, post 9, September 6, 2026

**Use case 1: Interpreter fuzzer**
Fuzzes extensions across CPython, PyPy, and RustPython. Currently must
attempt a build to discover implementation support. Machine-readable
declared support would eliminate this trial-and-error.

**Use case 2: Large-scale compatibility tester**
Downloads thousands of extensions, builds and runs test suites under
different implementations. Metadata would allow pre-filtering rather
than discovering support by building.

**Important caveat:** Daniel disclosed he works with the proposer.
These are real use cases but cannot be treated as independent validation.
Present them as a category of non-installer consumer need, not as
independent community endorsement.

---

## TIER 2 — Ambiguous (context only, do not lead with these)

### objgraph

Pure Python, gc module internals. Documented as CPython-focused.
Failure is at runtime. Not guaranteed to fail on PyPy in all scenarios.
Use only to illustrate that runtime-only failures exist as a category.

---

## TIER 3 — RETIRED

### RestrictedPython

**Do not use.** Eli Schwartz correctly identified the wheel is
incorrectly tagged. Ralf confirmed the downstream dependency case
is solved by PEP 508 markers. This example was conceded September 6.

---

## TIER 4 — Counterexamples (show analysis is balanced)

Include these when asked about scope to demonstrate the analysis
is not cherry-picking every package that mentions PyPy.

| Package | Why it is a counterexample |
|---|---|
| aiohttp | Falls back gracefully to pure-Python on PyPy |
| multidict | Implementation-specific build with fallback |
| coverage.py | PyPy triggers different build path, not a failure |
| python-zstandard | Implementation-dependent backend selection, not failure |

**Key message:** Implementation-dependent build behavior does NOT
automatically justify implementation metadata. guppy3 is the case
where existing mechanisms fail. Most packages are not guppy3.

---

## Research still needed

Find 3-5 more packages like guppy3 where:
- CPython classifier declared
- sdist published
- No PyPy wheels
- Build script detects non-CPython

**Search strategy:** PyPI packages with:
`Programming Language :: Python :: Implementation :: CPython`
AND published sdist AND no `pp*` wheel in any release.

Also reach out to:
- PyPy team — do they encounter packages where pre-build rejection
  would help? Would they use a `Supported-Implementation` field?
- GraalPy team — same question
- Any endorsement from an alternate implementation team is far stronger
  than finding more individual packages

---

## Classification framework

| Situation | Existing mechanism | New metadata needed? |
|---|---|---|
| CPython wheel exists | PEP 425 cp tag | No |
| PyPy wheel exists | PEP 425 pp tag | No |
| Generic pure-Python | py3-none-any | No (if correctly tagged) |
| Conditional dependency | PEP 508 marker | No |
| Builds everywhere, officially unsupported | Docs + classifiers | Unclear |
| sdist build requires CPython (guppy3) | No release-level mechanism | Potentially yes |
| Direct install of CPython-only package | No normative field | Potentially yes |
| Tooling across 1000s of packages | No standard field | Yes (Daniel's case) |