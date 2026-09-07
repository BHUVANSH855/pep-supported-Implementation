# PEP 794 Notes — Release-Level Core Metadata

**URL:** https://peps.python.org/pep-0794/
**Status:** Accepted
**Relevance:** High

PEP 794 adds `Import-Name` and `Import-Namespace` to Core Metadata.

The important structural precedent is:

```text
project/release fact
        ↓
Core Metadata
        ↓
same release-level information across artifacts
        ↓
potentially served by an index
```

PEP 794 explicitly explains that the metadata can be served by an index
independently of a wheel or sdist, and that the metadata describes the
project version rather than an individual artifact.

## Why it matters

This is a strong architectural precedent for asking whether implementation
support is also a release-level fact.

## What it does not prove

PEP 794 does not prove that implementation support belongs in Core Metadata.

It demonstrates that Core Metadata can contain standardized facts whose
meaning is broader than a single artifact.

Current conclusion:

> PEP 794 supports the feasibility of release-level Core Metadata, not the
> necessity of a new implementation-support field.
