# PEP 794 — Import Name Metadata

**URL:** https://peps.python.org/pep-0794/
**Status:** Accepted (September 2025)
**Author:** Brett Cannon
**Relevance:** High — structural precedent for adding project-level
Core Metadata fields

## What it does

Adds two new Core Metadata fields:

- `Import-Name` — the top-level import name of the package
- `Import-Namespace` — the namespace package the distribution belongs to

Both are project-level fields that can be served by an index
independently of individual artifacts.

## Why it matters for this proposal

PEP 794 explicitly rejected inferring project information from wheel
contents because:

1. It does not work for sdists
2. It requires inference rather than explicit project metadata
3. Project-level information belongs in Core Metadata, not wheel inspection

This is structurally identical to the argument for Requires-Implementation:

> Implementation compatibility is a project-level property that cannot
> be reliably inferred from wheel contents and is entirely absent from
> sdist artifacts.

## The Simple API connection

PEP 794's fields are served via the existing `data-core-metadata`
attribute on the Simple API (introduced by PEP 658 / PEP 714).

This means: any new Core Metadata field automatically participates
in the Simple API metadata transport. No new infrastructure needed.

## Key quote from PEP 794 motivation

The PEP argues that project-level Core Metadata allows tools to
"obtain the information without downloading every distribution artifact."

That is precisely the value proposition of Requires-Implementation
for sdists.

## Architectural pattern established by PEP 794

```
pyproject.toml
      |
      v
Core Metadata field
      |
      v
Simple API (data-core-metadata)
      |
      v
installer acts before downloading artifact
```

Requires-Implementation would follow this exact same pattern.
