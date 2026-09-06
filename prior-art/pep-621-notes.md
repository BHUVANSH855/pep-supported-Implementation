# PEP 621 Notes

## Purpose

PEP 621 defines how project metadata is declared in `pyproject.toml`.

It is important because a new standardized Core Metadata field would need a
corresponding project-level representation if projects are expected to
declare it statically.

## Relevant model

PEP 621 defines the:

```toml
[project]
```

table.

Fields defined by PEP 621 map to Core Metadata fields.

For example:

```toml
[project]
requires-python = ">=3.10"
```

maps to:

```text
Requires-Python: >=3.10
```

## Important constraint

Tools cannot arbitrarily add new standardized fields to `[project]`.

A new field requires a subsequent standards change.

Therefore a hypothetical:

```toml
[project]
supported-implementation = ["cpython"]
```

would require a specification defining:

1. the TOML field;
2. its mapping to Core Metadata;
3. its semantics;
4. its validation rules;
5. dynamic/static behavior;
6. interaction with existing metadata.

## Relationship to this research

PEP 621 establishes the project metadata layer that a future implementation
compatibility field would likely need to integrate with.

It does not provide such a field itself.

## Current conclusion

PEP 621 supports the feasibility of adding standardized project metadata,
but does not provide evidence that implementation compatibility should be
added.
