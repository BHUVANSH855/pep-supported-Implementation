# PEP 643 Notes — Metadata for Package Source Distributions

**URL:** https://peps.python.org/pep-0643/

**Status:** Final

**Relevance:** High

## What PEP 643 does

PEP 643 standardizes how Core Metadata is represented in source distributions and, importantly, defines how metadata values in an sdist relate to metadata in wheels subsequently built from that source.

The motivation was that metadata consumers historically could not reliably depend on the metadata available in source distributions and could therefore need to invoke the PEP 517 build machinery merely to obtain metadata. PEP 643 provides a standard mechanism for recording metadata in sdists while retaining support for fields whose values genuinely cannot be known until build time.

This is directly relevant to the proposed implementation-support research because it establishes that **release metadata can exist in an sdist before the source is built**.

## The `Dynamic` mechanism

PEP 643 adds a Core Metadata field:

```text id="3zv2kn"
Dynamic
```

When a field in an sdist is **not** marked as `Dynamic`, its value in a wheel built from that sdist MUST match the value in the sdist.

If a field is marked `Dynamic`, the wheel may contain any valid value for that field, including omitting it entirely.

Conceptually:

```text id="m7p5yq"
Static field:

sdist
  |
  +-- field = X
          |
          ↓
wheel built from sdist
  |
  +-- field MUST = X
```

whereas:

```text id="f8k3ra"
Dynamic field:

sdist
  |
  +-- field marked Dynamic
          |
          ↓
wheel
  |
  +-- value may be determined at build time
```

This distinction is essential for any future release-level metadata field.

## Why this matters for implementation support

Suppose a future Core Metadata specification introduced:

```text id="z6c1hw"
Supported-Implementation: cpython
```

If that field were defined as static metadata, PEP 643 provides an existing semantic framework under which:

```text id="9a4m0t"
sdist
    Supported-Implementation: cpython

        ↓ build

wheel
    Supported-Implementation: cpython
```

would be required to remain consistent.

That would make the support declaration a property of the **release metadata**, rather than an incidental property generated from the environment in which a particular wheel happened to be built.

This is exactly the distinction the research needs to investigate.

## Release metadata versus build-result metadata

The important conceptual boundary is:

```text id="r2m6cw"
Release metadata
    -> describes the distribution release

Build-result metadata
    -> may depend on the environment in which
       an individual artifact is produced
```

PEP 643 explicitly allows the latter through `Dynamic`.

A hypothetical implementation-support field therefore needs to answer:

```text id="c8w5vk"
Is implementation support a stable property
of the release?
```

or:

```text id="p4j9sx"
Can implementation support legitimately vary
between builds of the same source release?
```

If the intended semantics are release-level producer support, the field would normally need to behave like static metadata.

That conclusion is a design implication, not something PEP 643 itself establishes for implementation support.

## Why static metadata is useful for sdists

PEP 643's motivation is especially relevant to the proposed pre-build use case.

A consumer may have an sdist available and need metadata about the release before invoking the build backend.

If the relevant metadata is statically recorded in the sdist, the consumer can inspect it without first performing a source build. PEP 643 was specifically designed to make metadata in source distributions reliable enough for this purpose.

This gives the research a concrete pathway:

```text id="v5s7jh"
sdist
  ↓
static Core Metadata
  ↓
consumer decision
  ↓
build only if appropriate
```

A hypothetical implementation-support field could fit into this architecture **if** the field were justified independently.

## PEP 643 does not establish the need for the field

The existence of a mechanism for storing static metadata in an sdist does not imply that implementation support belongs in Core Metadata.

The logical relationship is:

```text id="d8k4pf"
PEP 643:
    Core Metadata can be reliably represented in sdists.

Research question:
    Is implementation support a Core Metadata fact
    worth representing there?
```

The second question remains unresolved.

PEP 643 therefore provides enabling infrastructure rather than evidence of necessity.

## Static support versus build-environment capability

A particularly important distinction is:

```text id="j4m7xs"
build environment:
    CPython

release support:
    CPython
```

These statements must not be conflated.

A wheel may have been built using CPython even though the resulting pure-Python distribution is intended to support PyPy.

Conversely, a project may build a generic wheel while intentionally declaring that it only supports CPython.

Therefore a hypothetical support field should not be inferred automatically from:

```text id="x6n2qp"
the interpreter used to build the wheel.
```

The value would need to come from the project's declared release semantics.

PEP 643's static/dynamic distinction provides a useful framework for keeping these concepts separate.

## Dynamic metadata is a potential complication

PEP 643 also prevents an overly simple assumption that every Core Metadata field is necessarily identical across all builds.

A field marked `Dynamic` may legitimately receive a different value in a wheel, because its value can depend on build-time information. Consumers are explicitly warned not to treat a dynamic value recorded in an sdist as canonical.

Therefore, if implementation support were standardized as Core Metadata, the specification would need to determine whether:

```text id="u2f6yb"
Supported-Implementation
```

could ever be dynamic.

If the intended semantics are a producer's release-level support policy, allowing the field to change merely because the build occurred on a different implementation could be dangerous.

For example:

```text id="z3q8cv"
Build A:
    CPython
    -> Supported-Implementation: cpython

Build B:
    PyPy
    -> Supported-Implementation: pypy
```

would describe different support policies for what is ostensibly the same release.

That would undermine the meaning of a release-level declaration unless explicitly intended.

This is therefore a significant design question.

## Relationship to PEP 621

PEP 621 provides the project-level authoring concept of `dynamic`, while PEP 643 provides the corresponding Core Metadata semantics for source distributions.

Together they establish a useful layered model:

```text id="0r6y9a"
Project authoring
    PEP 621
        ↓
Core Metadata
        ↓
PEP 643 sdist consistency rules
        ↓
wheel metadata
```

A future implementation-support declaration would need to fit coherently into this existing architecture.

However, neither PEP 621 nor PEP 643 establishes that such a declaration should exist.

## Relationship to `Requires-Python`

PEP 643 is also relevant to `Requires-Python`.

The PEP's development history explicitly considered whether `Requires-Python` needed special handling because of concerns about making important metadata static. The final design removed that special case by allowing fields to be dynamic generally while strongly encouraging backends to avoid dynamic metadata unless necessary.

This provides useful precedent for the present research:

```text id="w7c3mh"
Do not create special static/dynamic semantics
unless the field actually requires them.
```

Instead, the semantics of a future support field should be determined by its actual meaning.

## Pre-build inspection is technically plausible

PEP 643 therefore makes the following workflow technically plausible:

```text id="p1r6wt"
index / source distribution
        ↓
obtain sdist metadata
        ↓
inspect static release metadata
        ↓
make consumer decision
        ↓
invoke source build only if appropriate
```

This is important because one possible motivation for implementation-support metadata is to avoid attempting a source build on an implementation that the release explicitly does not support.

But there is a crucial additional requirement:

```text id="m8q4ys"
The consumer must actually have access to the
relevant metadata before the build decision.
```

PEP 643 establishes that reliable metadata can be stored in an sdist. It does not by itself guarantee that every index or resolver workflow exposes that metadata before downloading the sdist.

Therefore:

```text id="q5k2cn"
metadata can exist early
        !=
consumer can always obtain it early
```

The research must keep these questions separate.

## Residual-case implication

For an sdist-based implementation-support case to be strong evidence for a new field, the research should establish:

```text id="7x3wmp"
1. The exact release has a genuine implementation-support boundary.

2. The boundary is a property of the release rather than merely
   of one build environment.

3. The information can be represented as static release metadata.

4. Existing metadata does not already provide the required decision.

5. The consumer can obtain the relevant metadata early enough
   to avoid an unnecessary build.

6. The resulting decision provides meaningful practical benefit.
```

If these conditions are not satisfied, PEP 643 should not be used as evidence for the field.

## What PEP 643 does not prove

PEP 643 does **not** prove:

```text id="v2k7rx"
implementation support belongs in Core Metadata.
```

It does not define:

```text id="n8x4pf"
Supported-Implementation
```

and does not establish its semantics.

It proves something narrower and useful:

```text id="j5m1sd"
Core Metadata can contain reliable static information
in an sdist, with defined consistency behavior for
wheels built from that source.
```

That is enabling infrastructure.

## Current conclusion

PEP 643 establishes an important architectural capability:

```text id="h2q9wc"
static Core Metadata
        ↓
source distribution
        ↓
consistent wheel metadata
```

This makes a release-level implementation-support declaration technically plausible as **release metadata**, including for source distributions.

However, PEP 643 does not establish that implementation support is a required or appropriate Core Metadata field.

The correct research conclusion is therefore:

> **PEP 643 is enabling infrastructure, not evidence of necessity.**

If implementation-support metadata is eventually justified, PEP 643 provides an existing framework for determining whether the declaration is static or dynamic and for maintaining consistency between an sdist and wheels built from it.

The remaining question is entirely substantive:

```text id="b7v3nc"
Is there a recurring, consumer-useful release-level
implementation-support fact that existing metadata
cannot express adequately?
```

PEP 643 can help us represent such a fact if one is justified; it does not establish that the fact needs to exist.
