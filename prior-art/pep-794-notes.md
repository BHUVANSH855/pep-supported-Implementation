# PEP 794 Notes

## Purpose

PEP 794 is useful prior art for project/release-level metadata describing
properties of Python packages.

It proposes `Import-Name` and `Import-Namespace` metadata.

## Relevant idea

The important structural precedent is that metadata describing a project's
relationship to Python imports can be represented as standardized Core
Metadata.

This demonstrates that project-level facts can be standardized independently
of individual wheel filenames.

## Relationship to this research

A proposed implementation compatibility field would follow a similar
high-level model:

```text
project/release metadata
        ↓
Core Metadata
        ↓
available across distribution artifacts
```

The proposed field would differ in what it describes:

```text
PEP 794:
    import names / namespaces

Research proposal:
    Python implementation support
```

## Metadata transport

PEP 794 should not be treated as evidence that metadata is automatically
available without downloading artifacts.

For pre-download metadata access, the more directly relevant mechanisms are:

- PEP 658;
- PEP 714;
- the Simple Repository API.

These specifications define how repositories can expose Core Metadata
separately from distributions.

## Current conclusion

PEP 794 provides useful structural prior art for standardized project-level
Core Metadata.

It does not establish that implementation compatibility belongs in Core
Metadata, nor does it solve the pre-build implementation compatibility
problem by itself.
