# Real-World Package Cases

Cases where Python implementation compatibility is a genuine concern
at the project level, not just the artifact level.

---

## guppy3 — STRONG CASE (build-time)

**PyPI:** https://pypi.org/project/guppy3/
**Current version:** 3.1.7 (released May 11, 2026)
**Classifier declared:** Programming Language :: Python :: Implementation :: CPython

### What it does

Memory profiling tool for Python. Uses CPython-specific C internals.

### Implementation stance

PyPI page explicitly states:
> "This package is CPython only; PyPy and other Python implementations
> are not supported."

Also declares free-threaded CPython is not supported.

### Artifacts published

- `guppy3-3.1.7.tar.gz` (sdist)
- `guppy3-3.1.7-cp314-cp314-*.whl` (CPython-specific wheels)
- No PyPy wheels

### Build script evidence

`setup.py` contains:

```python
if sys.implementation.name != 'cpython':
    print(
        'setup.py: Warning: This guppy package only supports CPython.',
        'Compilation failure expected, but continuing anyways...'
    )
```

Then proceeds to compile two C extensions:
- guppy.sets.setsc
- guppy.heapy.heapyc

### Why this is the strongest case

1. The project explicitly declares CPython-only on PyPI
2. The build system itself detects non-CPython and expects failure
3. An sdist is published with no pre-build metadata signal
4. CPython-specific wheels exist so PyPy wheels will not match
5. A PyPy user receives the sdist and encounters a build failure
   with no normative metadata to have warned the installer in advance

### What we have NOT verified

We have not independently run `pypy -m pip install guppy3` and
captured the exact error output. Do not claim we reproduced the failure.
Cite the source code and PyPI declaration only.

---

## objgraph — MEDIUM CASE (runtime)

**PyPI:** https://pypi.org/project/objgraph/

### What it does

Draws Python object reference graphs. Uses gc module internals.

### Implementation concern

Pure Python but relies on gc module behavior that differs across
implementations. Documented as CPython-focused.

### Artifacts

Publishes py3-none-any wheel. Builds fine everywhere.
Failure is at runtime, not build time.

### Strength as evidence

Medium. The gc module exists in PyPy but behavior differs.
Not as clean as guppy3 because failure mode is not guaranteed
and may be fixable on the PyPy side.

---

## RestrictedPython — RETIRED EXAMPLE

**Do not use this example.**

Eli Schwartz correctly identified that RestrictedPython publishes a
py3-none-any wheel when it should publish a cpython-specific wheel.
The fix is a bug report to the project, not new metadata.

This example was conceded in the thread on September 6, 2026.

---

## Packages that DISPROVE the need for new metadata

These packages have implementation-specific builds but handle it
correctly through existing mechanisms. Include these in any
discussion to show the analysis is balanced.

### aiohttp
Has implementation-specific build paths. Falls back gracefully on PyPy.
Existing mechanisms work. No new metadata needed.

### multidict
Similar pattern. Implementation-specific build with fallback.

### coverage.py
PyPy triggers a different build configuration, not a failure.

### python-zstandard
Implementation-dependent backend selection, not a build failure.

---

## Classification framework

| Situation | Existing mechanism | New metadata needed? |
|---|---|---|
| CPython wheel | PEP 425 cp tag | No |
| PyPy wheel | PEP 425 pp tag | No |
| Generic pure-Python | py3-none-any | No |
| Conditional dependency | PEP 508 marker | No |
| Builds everywhere, officially unsupported | Docs + classifiers | Unclear |
| sdist build itself requires CPython | No release-level mechanism | Potentially yes |
| Release should be skipped by version | Requires-Python | No |
| Release should be skipped by implementation | No equivalent field | Potentially yes |

---

## Research needed

Find 5-10 real packages where:
- The source build itself requires a specific Python implementation
- A compatible wheel is unavailable for the user's implementation
- The incompatibility is statically knowable before attempting the build

guppy3 is one confirmed case. More needed before the proposal is
strong enough to proceed past community objections.
