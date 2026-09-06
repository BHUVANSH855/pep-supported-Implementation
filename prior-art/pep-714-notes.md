# PEP 714 Notes

## Purpose

PEP 714 updates the naming used by PEP 658 for Core Metadata in the Simple
Repository API.

## Relevant names

For HTML Simple API responses:

```text
data-core-metadata
```

For the JSON representation:

```text
core-metadata
```

## Relationship to this research

PEP 714 is relevant because a future Core Metadata field does not need a new
repository transport protocol merely because the metadata field is new.

If a repository exposes the Core Metadata document, a new standardized field
could be carried through the existing mechanism.

## Important limitation

PEP 714 does not guarantee that repositories expose metadata sidecars.

Therefore:

```text
Core Metadata field exists
```

does not automatically imply:

```text
installer can always inspect field before downloading artifact
```

## Current conclusion

PEP 714 strengthens the case that a new Core Metadata field could reuse
existing metadata transport infrastructure.

It does not establish that the field is necessary.
