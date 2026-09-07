# Simple API Core Metadata

The Simple Repository API can expose Core Metadata associated with a
distribution.

Relevant attributes:

```text
data-core-metadata
core-metadata
```

depending on the representation.

References:

- PEP 658: https://peps.python.org/pep-0658/
- PEP 714: https://peps.python.org/pep-0714/

## Research relevance

A release-level Core Metadata field could potentially be inspected before the
distribution is downloaded, when the repository provides the metadata.

Conceptually:

```text
index
  ↓
Core Metadata
  ↓
implementation support
  ↓
candidate filtering
```

## Limitation

The metadata sidecar is optional.

Therefore:

```text
standardized field
```

does not imply:

```text
field always available before download
```

A future design would need to specify fallback behavior.
