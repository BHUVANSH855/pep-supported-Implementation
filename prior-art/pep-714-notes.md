# PEP 714 Notes — Simple API Core Metadata Naming

**URL:** https://peps.python.org/pep-0714/

**Status:** Final

**Relevance:** Medium/High

## What PEP 714 does

PEP 714 renames the Simple API attributes used to expose the separately served Core Metadata introduced by PEP 658.

For the HTML representation of the Simple API, the current attribute is:

```text id="k2v9fd"
data-core-metadata
```

For the JSON representation, the current key is:

```text id="p5x1zw"
core-metadata
```

PEP 714 explicitly states that the supported values remain the same; the change is primarily a correction and alignment of the naming between the HTML and JSON representations.

## Relationship to PEP 658

The architectural relationship is:

```text id="n7c4mq"
PEP 658
    ↓
defines separate Core Metadata transport

PEP 714
    ↓
renames the transport attributes
```

PEP 714 does not replace the underlying PEP 658 model.

It preserves the idea that a repository can expose a distribution's Core Metadata independently so that a client can inspect it without downloading the complete artifact.

## Current HTML representation

For HTML Simple API pages, a repository supporting the mechanism uses:

```text id="q8r3xs"
data-core-metadata
```

when the separately served Core Metadata is available.

Clients are required to read this current attribute when present. The previous `data-dist-info-metadata` name is retained only as an optional compatibility mechanism when the new attribute is absent.

## Current JSON representation

For JSON Simple API pages, the corresponding key is:

```text id="v6m1yc"
core-metadata
```

Clients consuming the JSON representation must read this key when present. The legacy `dist-info-metadata` key may optionally be supported when the new key is absent.

This gives current packaging research a stable vocabulary:

```text id="j9k4wp"
HTML:
    data-core-metadata

JSON:
    core-metadata
```

rather than the original PEP 658 names.

## Why this matters to implementation-support research

If a new implementation-support field were eventually added to Core Metadata, PEP 714 means there is already an established repository transport path through which that field could be exposed.

Conceptually:

```text id="s2q7nx"
Core Metadata
    |
    +-- existing fields
    |
    +-- hypothetical implementation-support field
            ↓
        Simple API
            ↓
        data-core-metadata / core-metadata
            ↓
        resolver
```

No new repository transport mechanism would necessarily be required merely because a new Core Metadata field existed.

This is useful architectural prior art.

## Transport does not define semantics

PEP 714 does not define the meaning of individual Core Metadata fields.

It defines the names used to expose the metadata transport introduced by PEP 658.

Therefore the following are separate questions:

```text id="w4p8kc"
Can metadata be transported?
        ->
PEP 658 / PEP 714

What does a metadata field mean?
        ->
Core Metadata specification

How is that field authored?
        ->
project metadata / build backend specifications

What decision does a resolver make from it?
        ->
consumer/tool semantics
```

A hypothetical:

```text id="c8m2zr"
Supported-Implementation: cpython
```

would require its own Core Metadata semantics before PEP 714 could transport it.

## Optional availability remains important

PEP 714 preserves the optional nature of the PEP 658 mechanism.

A repository may not expose separately served Core Metadata. In that situation, clients cannot assume that the metadata is available through the Simple API before downloading the distribution.

Therefore:

```text id="g7q5vx"
new Core Metadata field
        +
PEP 714 transport
```

does not imply:

```text id="m3n9ka"
every repository exposes that field
before artifact download
```

This limitation must remain explicit in any proposed consumer-benefit argument.

## PEP 714 and resolver behavior

PEP 714 does not itself specify that a resolver must make a new kind of candidate-selection decision from arbitrary Core Metadata.

It specifies how the metadata transport is named and consumed.

Therefore, even if:

```text id="z6r1wp"
Supported-Implementation
```

were added to Core Metadata, additional work would still be required to establish:

```text id="f8c3mq"
- how resolvers interpret it;
- whether it affects candidate eligibility;
- whether it is advisory or normative;
- what happens when it conflicts with other metadata;
- whether unsupported releases should be skipped or rejected.
```

This is a critical boundary.

Adding a field to Core Metadata does not automatically cause installers to use it as a candidate-selection constraint.

## Relationship to PEP 658's original names

For historical research, the mapping is:

```text id="r2v7kh"
PEP 658 / original HTML:
    data-dist-info-metadata

PEP 714 / current HTML:
    data-core-metadata
```

and:

```text id="m8q4jp"
PEP 658 / original JSON:
    dist-info-metadata

PEP 714 / current JSON:
    core-metadata
```

The semantic role is the same; PEP 714 standardizes the corrected names.

This distinction should be retained when reading older documentation or historical implementation discussions.

## Current conclusion

PEP 714 is **transport prior art**, not implementation-support prior art.

It demonstrates that the packaging ecosystem already has standardized mechanisms for exposing Core Metadata through both HTML and JSON Simple API representations:

```text id="v4s9qn"
HTML:
    data-core-metadata

JSON:
    core-metadata
```

These mechanisms can potentially make a future Core Metadata field visible to clients before the associated artifact is downloaded, where repository support exists.

But PEP 714 does not define:

```text id="p1z6xc"
Supported-Implementation
```

and does not establish that such a field should influence candidate selection.

Therefore:

> **PEP 714 demonstrates existing transport infrastructure, not a need for implementation-support metadata.**

The research must still establish the missing semantic and the concrete consumer decision that would justify it.
