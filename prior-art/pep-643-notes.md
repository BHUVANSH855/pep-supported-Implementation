# PEP 643 Notes — Metadata for Package Source Distributions

**URL:** https://peps.python.org/pep-0643/
**Status:** Final
**Relevance:** High

PEP 643 establishes static Core Metadata semantics for source distributions.

A field that is not marked dynamic must have consistent metadata between the
sdist and wheels built from that source.

## Why this matters

A release-level field such as a hypothetical:

```text
Supported-Implementation: cpython
```

could therefore be treated as release metadata rather than build-result
metadata.

That makes pre-build inspection technically plausible.

## What PEP 643 does not prove

It does not prove that implementation support belongs in Core Metadata.

It only establishes that static release metadata in sdists can be defined
with consistency requirements.

## Current conclusion

PEP 643 is enabling infrastructure, not evidence of necessity.
