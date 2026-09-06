# Real-world cases

Packages and tools that illustrate the gap this proposal addresses.

---

## guppy3

**PyPI:** https://pypi.org/project/guppy3/
**Current release:** 3.1.7

guppy3 is a memory profiling tool that uses CPython-specific C internals.
Its PyPI page states explicitly:

> "This package is CPython only; PyPy and other Python implementations
> are not supported."

It publishes CPython-specific wheels (`cp314-cp314-*.whl`) and a source
distribution (`guppy3-3.1.7.tar.gz`). No PyPy wheels are published.

Its `setup.py` contains:

```python
if sys.implementation.name != 'cpython':
    print(
        'setup.py: Warning: This guppy package only supports CPython.',
        'Compilation failure expected, but continuing anyways...'
    )
```

A user on PyPy or GraalPy would receive the sdist (no compatible wheel
exists) and encounter a build failure. There is currently no pre-build
metadata signal in the sdist that would allow an installer to reject
this candidate before attempting the build.

---

## Non-installer tooling

Some tooling works across multiple Python implementations at scale and
needs machine-readable implementation support information:

- Tools that fuzz extensions across CPython, PyPy, and RustPython
  currently must attempt a build to discover whether a given extension
  supports a given implementation.

- Systems that download large numbers of packages and run their test
  suites under different implementations would benefit from pre-filtering
  by declared support rather than discovering compatibility by building.

For these consumers, Trove classifiers are insufficient because they
are not structured for programmatic querying of implementation support
across thousands of packages in a consistent way.

---

## Packages where existing mechanisms are sufficient

Implementation-specific build behaviour does not automatically mean
new metadata is needed. Many packages handle it well already:

- **aiohttp** — falls back gracefully to a pure-Python implementation
  on PyPy; builds successfully on multiple implementations.
- **multidict** — implementation-specific build with a working fallback.
- **coverage.py** — PyPy triggers a different build path, not a failure.
- **python-zstandard** — implementation-dependent backend selection.

The gap is specifically for packages like guppy3 where the incompatibility
is fundamental and statically knowable, not for packages that adapt their
build to the environment.

---

## packaging.tags issue #311

https://github.com/pypa/packaging/issues/311

`packaging.tags` does not generate `cp3-none-any` tags for pure-Python
CPython-specific packages. Only `pp3-none-any` was added as a special
case. This means the "just use an implementation-specific wheel tag"
answer is not fully supported by current tooling for pure-Python packages
that are CPython-only.