# PEP 658 Notes

## Purpose

PEP 658 defines a mechanism for repositories to serve Core Metadata
separately from distribution files.

## Core idea

A Simple API repository can expose metadata associated with a distribution
without requiring the client to download the full distribution first.

This is directly relevant to compatibility metadata.

If implementation support were represented in Core Metadata, PEP 658 provides
a mechanism through which tooling could potentially inspect that declaration
before downloading or building the distribution.

## Important limitation

The metadata sidecar is optional.

Therefore a client cannot assume that every repository will expose Core
Metadata separately.

A proposed compatibility field must not assume universal pre-download
metadata availability.

## Relationship to sdists

PEP 658 is especially relevant to the research question because the desired
workflow is conceptually:

```text
repository
    ↓
Core Metadata
    ↓
implementation compatibility decision
    ↓
download/build only if appropriate
```

rather than:

```text
repository
    ↓
download sdist
    ↓
build sdist
    ↓
discover incompatibility
```

## Current conclusion

PEP 658 provides an existing transport mechanism for standardized metadata.

It does not define implementation compatibility semantics itself.
