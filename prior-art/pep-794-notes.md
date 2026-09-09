# PEP 794 Notes — Release-Level Core Metadata

**URL:** https://peps.python.org/pep-0794/

**Status:** Accepted

**Relevance:** High

## What PEP 794 does

PEP 794 adds two repeatable Core Metadata fields:

```text id="4z6w2p"
Import-Name
Import-Namespace
```

It also adds corresponding project-level fields:

```toml id="x8q1mv"
[project]
import-names = [...]
import-namespaces = [...]
```

The PEP introduced these fields to provide standardized information about the import names a project provides when installed. ([peps.python.org](https://peps.python.org/pep-0794/))

## The important structural precedent

The most relevant part of PEP 794 is its treatment of the metadata as a property of the **project release**, rather than of one particular wheel or sdist.

PEP 794 requires the relevant information to be consistent across the sdists and wheels belonging to the same project release. The PEP explicitly states that this means the information is not specific to the individual distribution artifact in which it appears, but to the release version to which the artifact belongs. ([peps.python.org](https://peps.python.org/pep-0794/))

Conceptually:

```text id="n6r4tx"
project/release fact
        ↓
Core Metadata
        ↓
same release-level information
across sdists and wheels
        ↓
potentially available to metadata consumers
```

This is strong architectural prior art for the present research.

## Why this matters

The proposed research asks whether Python implementation support can be a similar kind of release-level fact.

Conceptually:

```text id="w3k7qp"
PEP 794:

    What import names does this release provide?

Research proposal:

    Which Python implementations does this release support?
```

Both questions potentially concern properties of a release rather than properties of an individual artifact.

That makes PEP 794 particularly useful when evaluating whether implementation support could legitimately belong in Core Metadata.

## Release fact versus artifact fact

PEP 794 establishes an important distinction:

```text id="f2q8mc"
artifact-specific information
        !=
release-level information
```

For example, two wheels for the same release may differ in:

```text id="v7m3ka"
Python tag
ABI tag
platform tag
```

while still carrying the same `Import-Name` information because they belong to the same project release. ([peps.python.org](https://peps.python.org/pep-0794/))

This is structurally similar to the distinction being investigated for implementation support.

A hypothetical:

```text id="r5x9nw"
Supported-Implementation
```

could likewise be defined as a property of the release rather than of an individual wheel.

## This does not mean implementation support belongs in Core Metadata

PEP 794 establishes **feasibility and precedent**, not necessity.

The correct inference is:

```text id="b8m2jq"
PEP 794 demonstrates:
    Core Metadata can contain release-level facts
    that apply consistently across artifacts.

Research question:
    Is implementation support such a fact,
    and is it valuable enough to standardize?
```

The second question remains unresolved.

PEP 794 therefore cannot by itself justify adding another Core Metadata field.

## Relationship to wheel tags

PEP 794 is particularly useful because it demonstrates that release-level metadata can coexist with artifact-specific compatibility information.

For example:

```text id="p6c4yr"
Release:
    Import-Name = foo

Wheel A:
    cp313-cp313-manylinux...

Wheel B:
    cp313-cp313-win_amd64
```

The wheels have different artifact compatibility properties while belonging to the same release and sharing the release-level import metadata.

This reinforces the conceptual separation:

```text id="m9w5ks"
Core Metadata
    ->
facts about the distribution/release

Wheel tags
    ->
compatibility of an individual built artifact
```

A future implementation-support field could fit into that distinction if its semantics are genuinely release-level.

## Relationship to PEP 643

PEP 643 provides the relevant consistency model for static metadata in source distributions.

PEP 794 then provides a concrete example of a new Core Metadata field whose value is intended to describe the release rather than one artifact.

Together:

```text id="x2v8hm"
PEP 643
    ->
static/release metadata consistency

PEP 794
    ->
actual release-level Core Metadata field
```

This combination is stronger architectural precedent than either PEP alone.

It demonstrates that a new release-level fact can be represented consistently across:

```text id="y7q1nc"
sdist
wheel
other distribution artifacts
```

without making it an artifact-specific compatibility tag.

## Relationship to PEP 658 and PEP 714

PEP 658 and PEP 714 provide transport mechanisms for Core Metadata through the Simple Repository API.

Therefore the architectural chain can be:

```text id="a4k7qp"
release-level fact
        ↓
Core Metadata
        ↓
PEP 658 / PEP 714 transport
        ↓
repository/index
        ↓
metadata consumer
```

This is relevant because a release-level implementation-support field would be useful only if consumers could obtain and act on it.

However, transport availability remains separate from semantic justification.

The fact that PEP 794 metadata can be transported does not establish that implementation support needs the same treatment.

## Interaction with project metadata

PEP 794 also establishes project-level authoring keys:

```toml id="h3n8wp"
[project]
import-names = [...]
import-namespaces = [...]
```

This is relevant prior art for the possible authoring shape of a future support declaration.

If implementation support were eventually standardized, a corresponding project-level field could be considered.

But the logical order remains:

```text id="c6q2rz"
semantic requirement
        ↓
Core Metadata definition
        ↓
project-level authoring representation
```

not:

```text id="k8m4vf"
convenient TOML key
        ↓
therefore new Core Metadata field
```

## Important release-consistency question

PEP 794 raises an important design question for implementation support:

```text id="p7x5mc"
Can support differ between artifacts
of the same release?
```

If the answer is no, then implementation support is naturally modeled as release-level metadata.

If the answer is yes, then an ordinary release-level Core Metadata field may be the wrong abstraction.

For example, suppose:

```text id="q3n6wb"
same release
    |
    +-- wheel A supports CPython only
    |
    +-- wheel B supports PyPy only
```

That could represent an artifact compatibility distinction rather than a single release-level support policy.

The research must therefore ensure that empirical examples actually demonstrate a **release-level** boundary.

## PEP 794 as a residual-case test

A candidate case should be stronger if the evidence shows:

```text id="r8m2vq"
same release
    ↓
multiple artifacts
    ↓
artifact compatibility may differ
    ↓
producer support policy remains the same
    ↓
support policy cannot be represented adequately
    by existing artifact mechanisms
```

That is the kind of situation where PEP 794's release-level metadata model becomes genuinely relevant.

By contrast, if the only difference is:

```text id="j5q9kx"
wheel A is CPython-specific
```

then wheel tags may already solve the problem.

## Current conclusion

PEP 794 provides **strong architectural precedent for release-level Core Metadata**.

It demonstrates that Core Metadata can contain standardized facts that:

```text id="v4n7qs"
- describe a project release;
- remain consistent across the release's sdists and wheels;
- are not properties of one individual artifact;
- can participate in existing metadata transport mechanisms.
```

However, it does not establish that implementation support belongs in Core Metadata.

The correct conclusion is:

> **PEP 794 supports the feasibility and architectural legitimacy of release-level Core Metadata, but it does not establish the necessity of a new implementation-support field.**

The research still needs to demonstrate that implementation support is:

```text id="w9c2mp"
1. genuinely release-level;
2. distinct from artifact compatibility;
3. not already expressible by existing metadata;
4. useful to a concrete consumer;
5. stable enough to standardize.
```

PEP 794 should therefore be treated as **strong structural prior art**, not as evidence that the proposed field is required.
