# PEP 643 — Metadata for Package Source Distributions

**URL:** https://peps.python.org/pep-0643/
**Status:** Final
**Relevance:** Medium — establishes that sdist metadata can be trusted
for static fields

## What it does

Standardises reliable Core Metadata in source distributions by
introducing the `Dynamic` field. A field listed as Dynamic may
differ from what the final built distribution declares. A field
NOT listed as Dynamic must have the same value in the sdist as
in any wheel built from it.

## Why it matters for this proposal

PEP 643 makes the following architectural claim credible:

> Static Core Metadata fields in an sdist can be trusted by
> consumers without executing the build system.

This is the foundation for using an sdist's Core Metadata to make
pre-build compatibility decisions.

Without PEP 643, requiring sdist metadata to be authoritative would
be architecturally unsound. With it, a static Requires-Implementation
field in an sdist would be a valid, trustworthy signal.

## Implication

If Requires-Implementation is added to Core Metadata, it should be
a non-Dynamic field — meaning it must have the same value in the
sdist as in any wheel built from it. This is consistent with how
Requires-Python works today.
