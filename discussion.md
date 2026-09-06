# Discussion log

Summary of the pre-PEP Discourse thread and related discussions.

Thread: https://discuss.python.org/t/pre-pep-requires-implementation-declaring-python-implementation-compatibility-in-core-metadata/108898

---

## Key points raised and responses

**Paul Moore (CPython core developer, packaging PEP delegate)** asked
whether there are use cases beyond the sdist build-failure case, and
suggested exploring PEP 725 first.

After reviewing PEP 725, the build-failure case (where an sdist cannot
be built on a given implementation) may fit PEP 725's `build-requires`
model. However, PEP 725's `build-requires` and `host-requires` fields
have `Core Metadata: N/A`, so they do not produce a signal that an
installer can act on before attempting a build.

Paul also confirmed directly:

> "Currently, no I don't think there is [a standard way for an installer
> to know a package is CPython-only from sdist metadata before building]."

**Eli Schwartz** correctly pointed out that the RestrictedPython example
used initially was not a good one — if a package is CPython-only, it
should publish a CPython-specific wheel rather than `py3-none-any`. That
is a packaging bug in the specific project, not a gap in the standard.
The `packaging.tags` library also has a known issue (#311) where
`cp3-none-any` tags are not generated, though `pp3-none-any` was added.

**Ralf Gommers (PEP 725 co-author, NumPy/SciPy)** raised two points:

1. The staleness concern: if a package declares it does not support PyPy,
   and PyPy later gains compatibility, the metadata becomes harmful.
   This is a legitimate concern and the reason the proposal has shifted
   toward a positive "known to support" framing rather than an exclusionary
   "does not support" constraint.

2. PEP 725 is in final stages awaiting the Packaging Steering Council
   (post #98). The runtime implementation compatibility case was not
   addressed by PEP 725 and appears outside its current scope.

**Daniel Diniz** noted that a positive assertion ("these implementations
are known to work") is more stable and honest than a negative one, and
described two tooling use cases: fuzzing extensions across multiple
implementations, and large-scale compatibility testing across thousands
of packages. Both currently require attempting a build to discover
implementation support; machine-readable metadata would allow pre-filtering.

He also raised the important design question: if a release declares
`cpython` as supported and `pypy` is absent, does that mean PyPy is
unsupported, or only that PyPy support is unknown? The proposed answer
is the latter — absence means no claim, not incompatibility.

---

## Open design questions

See `design/open-questions.md` for the full list. The central unresolved
question is the absence semantics: what should a tool infer when a
particular implementation is not listed?

---

## Related thread

A comment was posted in the PEP 725 discussion (post #96) asking whether
the runtime implementation compatibility case is in scope for PEP 725:

https://discuss.python.org/t/pep-725-specifying-external-dependencies-in-pyproject-toml-round-2/103890/96

No direct response has been received. Based on post #98 from Ralf
Gommers, PEP 725 is in final stages and not being extended for this use case.