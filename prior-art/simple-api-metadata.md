# Simple API Metadata

## Purpose

The Simple Repository API provides package indexes with a standardized way to
list distribution files.

Core Metadata can optionally be exposed alongside distribution links.

## Relevant metadata

The current Simple API uses:

```text
data-core-metadata
```

for HTML responses.

The JSON representation uses:

```text
core-metadata
```

## Why this matters

The research problem can be represented as:

```text
package index
    ↓
release metadata
    ↓
implementation compatibility
    ↓
candidate selection
```

rather than requiring:

```text
download sdist
    ↓
build
    ↓
discover incompatibility
```

## Limitation

Core Metadata exposure is optional.

Therefore the existence of a standardized field would not automatically make
pre-download filtering universally possible.

Tools would still need a fallback path.

## Relationship to PEP 658 and PEP 714

PEP 658 introduced the metadata-serving mechanism.

PEP 714 standardized the current naming of the metadata attributes.

These specifications should therefore be considered together when evaluating
the practical benefits of a new Core Metadata field.
