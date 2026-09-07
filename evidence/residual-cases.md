# Residual Cases

This file contains the most useful real-world releases found so far.

A **residual case** is not merely a package that says “CPython only”.
For this research, a strong residual case has several of these properties:

1. the producer explicitly declares an implementation restriction;
2. the release contains an sdist or otherwise presents a release-level
   candidate;
3. the published wheel is implementation-generic (`py3-...`) or does not
   otherwise encode the restriction;
4. `Requires-Python` cannot express the restriction;
5. PEP 508 dependency markers do not express the package's own support;
6. the restriction is not simply an ABI feature that belongs to PEP 780;
7. the restriction is not safely inferable from the absence of a wheel;
8. a machine-readable release-level declaration could change a pre-install
   decision.

The cases below are evidence, not proof that a new field is necessary.

---

## Case A — RestrictedPython 8.5

### Published release facts

PyPI lists:

```text
Requires-Python:
    >=3.10,<3.16

Source:
    restrictedpython-8.5.tar.gz

Wheel:
    restrictedpython-8.5-py3-none-any.whl

Classifier:
    Programming Language :: Python :: Implementation :: CPython
```

Source: https://pypi.org/project/RestrictedPython/8.5/

The project description explicitly states that RestrictedPython supports only
CPython and not PyPy or other Python implementations.

### Why this is strong

The wheel is:

```text
py3-none-any
```

The current packaging specification defines `py` as generic Python rather than
a particular implementation.

PEP 425 describes `py` as “Generic Python (does not require
implementation-specific features)”.

Source:
https://peps.python.org/pep-0425/

Therefore the release contains two different statements:

```text
Artifact:
    no implementation-specific wheel requirement

Producer:
    CPython-only support
```

### Existing mechanisms

| Mechanism | Can it express the producer's restriction? |
|---|---|
| `Requires-Python` | No — only version |
| PEP 508 markers | No — they condition dependencies, not the distribution itself |
| wheel tag | No — `py3-none-any` is intentionally generic |
| Trove classifier | Yes, descriptively |
| PEP 780 | No — not an ABI-feature problem |
| PEP 825 | Not at release level; it concerns wheel variants |
| No-sdist-build policy | Could avoid some source builds, but does not express the support fact |

### Residual status

**Strong residual case.**

The main unresolved question is whether the classifier should remain
descriptive or whether this producer assertion deserves a normative,
machine-actionable representation.

---

## Case B — HAX 0.3.0

### Published release facts

PyPI lists:

```text
hax-0.3.0.tar.gz
hax-0.3.0-py3-none-any.whl
```

and says HAX supports CPython 3.7+ on all platforms.

Source:
https://pypi.org/project/hax/

The project source contains:

```python
if implementation.name != "cpython":
    raise RuntimeError("HAX only supports CPython!")

if version_info < (3, 7):
    raise RuntimeError("HAX only supports Python 3.7+!")
```

Source:
https://github.com/brandtbucher/hax/blob/add83a96a13458d66c42a8f58860e8fc25520fe4/hax/_checks.py

### Why this is stronger than a classifier-only case

The restriction is actively enforced at runtime.

This gives:

```text
documentation
+
published artifact
+
source-level runtime enforcement
```

### Existing mechanisms

`Requires-Python` can represent the version boundary:

```text
>=3.7
```

but cannot represent:

```text
implementation == cpython
```

The wheel tag is `py3-none-any`, so it does not encode CPython-only
compatibility.

### Residual status

**Strong residual case.**

It is particularly useful because it is a pure-Python artifact whose
implementation restriction is semantic rather than ABI-driven.

---

## Case C — Likepy 0.3.0

### Published release facts

PyPI lists:

```text
Requires-Python: >=3.6,<3.12

likepy-0.3.0.tar.gz
likepy-0.3.0-py3-none-any.whl

Programming Language :: Python :: Implementation :: CPython
```

The project description says:

> Likepy only supports CPython. It does not support PyPy and other Python
> implementations.

Source:
https://pypi.org/project/likepy/

### Existing mechanisms

The Python version restriction is covered by `Requires-Python`.

The implementation restriction is not encoded in the `py3-none-any` wheel.

The classifier communicates the producer's position descriptively.

### Residual status

**Strong residual case**, although the project is older than the 2026 cases.

It is useful because it independently reproduces the same artifact/support
mismatch.

---

## Case D — simple-ctx-log 0.0.3

### Published release facts

PyPI says the package uses `sys._getframe` and lists the limitation:

```text
Uses sys._getframe (CPython only)
```

The release publishes:

```text
simple_ctx_log-0.0.3.tar.gz
simple_ctx_log-0.0.3-py3-none-any.whl
```

Source:
https://pypi.org/project/simple-ctx-log/

Released:
February 1, 2026.

### Why it matters

This is a recent independent example of:

```text
pure Python
+
implementation-specific semantics
+
py3-none-any
+
sdist
```

### Residual status

**Strong residual candidate.**

The repository should avoid claiming more than the published evidence:
the PyPI description establishes the restriction, but this case has not yet
been independently validated with a source-level runtime guard.

---

## Case E — TribeCore 4.7.3

### Published release facts

PyPI says:

```text
CPython only.
PyPy and other Python implementations are not supported.
```

It publishes an sdist and wheels such as:

```text
tribecore-4.7.3-py3-none-win_amd64.whl
tribecore-4.7.3-py3-none-manylinux2014_x86_64.whl
tribecore-4.7.3-py3-none-macosx_13_0_universal2.whl
```

Source:
https://pypi.org/project/tribecore/

Released:
August 6, 2026.

The project explicitly explains that `py3-none-{platform}` is used so one
wheel works across Python 3.x versions on a given platform.

### Why it matters

This demonstrates that the phenomenon is not limited to
`py3-none-any`.

The Python implementation component can remain generic even when the wheel is
platform-specific.

### Residual status

**Strong candidate**, with an important caveat: the project uses bundled
native libraries, so the source/build and artifact layers deserve separate
analysis.

---

## Case F — winuvloop 0.2.5

### Published release facts

PyPI publishes:

```text
winuvloop-0.2.5.tar.gz
winuvloop-0.2.5-py3-none-any.whl
```

Released:
August 24, 2026.

Source:
https://pypi.org/project/winuvloop/

The package is a wrapper around platform-specific event-loop implementations.
Its compatibility should therefore be analyzed through its dependencies rather
than inferred from the wrapper wheel alone.

### Why it is useful

It is a good test of **support inheritance**:

```text
package A
    ↓
depends on implementation-specific package B
```

A resolver must not automatically infer:

```text
B supports CPython only
        ↓
A is CPython only
```

because A could provide fallbacks or conditionally select different
dependencies.

### Residual status

**Mixed / requires dependency-graph analysis.**

Do not use this as a clean proof case until the complete metadata graph has
been inspected.

---

# Control cases

## psutil

psutil contains implementation-specific build logic, but it supports multiple
Python implementations.

This is an important warning:

```text
implementation-specific source code
    ≠
implementation unsupported
```

The research must not classify every occurrence of
`sys.implementation` as a restriction.

## Autobahn

Autobahn supports multiple Python implementations while some optional/native
components have narrower implementation support.

This demonstrates:

```text
CPython-only dependency/component
    ≠
CPython-only release
```

Therefore implementation support cannot safely be inferred transitively.

## Guppy3

Guppy3 is a strong implementation/ABI case, but its CPython-specific wheels
already encode substantial artifact-level information.

It is therefore valuable mainly for studying the boundary between:

```text
implementation identity
ABI configuration
Python version
artifact compatibility
```

rather than as the cleanest residual case.

## bocpy

bocpy is a strong CPython/private-ABI example. Its compatibility depends on
CPython-specific private APIs and Python-version details.

It demonstrates why:

```text
Supported-Implementation: CPython
```

would not replace Python-version or ABI metadata.

---

# Current residual-case conclusion

The strongest repeated pattern is:

```text
producer:
    CPython only

wheel:
    py3-...

sdist:
    present

Requires-Python:
    expresses version only

PEP 508:
    expresses dependency applicability, not self-compatibility

classifier:
    descriptive support signal

normative release-level implementation constraint:
    absent
```

This is a real phenomenon.

Whether it justifies a new standard remains open.
