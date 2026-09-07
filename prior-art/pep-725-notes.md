# PEP 725 Notes

**URL:** https://peps.python.org/pep-0725/
**Status:** Draft / active discussion
**Relevance:** High

PEP 725 proposes standardized external dependency metadata in `pyproject.toml`
under `[external]`.

It distinguishes:

```text
build-requires
host-requires
dependencies
```

and explicitly considers cross-compilation.

## Why it matters here

A package might require a particular Python implementation **to build**.

That is different from:

```text
the resulting release supports a particular implementation at runtime
```

The research must keep these concepts separate.

## Current specification boundary

PEP 725 does not currently define CPython/PyPy/etc. as the implementation
support vocabulary being investigated here.

It therefore does not directly replace a release-level implementation-support
declaration.

## Important caution

The research should not say:

> PEP 725 cannot solve implementation requirements.

The correct statement is:

> PEP 725 is a relevant build/host dependency mechanism and may be extended
> or combined with other mechanisms; its current scope is not equivalent to
> release-level implementation support.

## Current conclusion

PEP 725 is a major design dependency and alternative to investigate, not an
irrelevant proposal.
