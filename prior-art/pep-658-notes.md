# PEP 658 Notes — Serve Distribution Metadata in the Simple Repository API

**URL:** https://peps.python.org/pep-0658/

**Status:** Accepted

**Relevance:** High

## What PEP 658 does

PEP 658 defines a mechanism for repositories using the Simple Repository API to expose a distribution's Core Metadata separately from the distribution file itself.

A repository can indicate that metadata is independently available for a particular distribution. A client can then fetch the metadata without downloading the complete wheel or source distribution.

The mechanism was motivated by packaging workflows that inspect metadata from multiple candidate distributions in order to select an appropriate candidate. Without separately served metadata, tools may need to download distributions that they ultimately do not install, creating unnecessary network and processing cost.

## The important workflow

PEP 658 makes the following workflow possible:

```text id="8t6nqk"
Simple Repository API
        ↓
distribution metadata
        ↓
candidate evaluation
        ↓
download artifact only if appropriate
```

For the implementation-support research, this creates a technically plausible path:

```text id="6c4v2s"
index
  ↓
Core Metadata
  ↓
implementation-support decision
  ↓
download/build only if appropriate
```

This is potentially significant for source distributions because the consumer may be able to inspect release metadata without first downloading and building the sdist.

## PEP 658 applies to sdists as well as wheels

PEP 658 specifies that the independently served metadata must be the distribution's canonical Core Metadata and explicitly applies the mechanism to standards-compliant wheels and source distributions.

Therefore the mechanism is not merely a wheel optimization.

Conceptually:

```text id="z9s4jx"
wheel
    ↓
Core Metadata

sdist
    ↓
Core Metadata
```

can both participate in the same metadata transport mechanism.

This matters because one potential motivation for implementation-support metadata is avoiding a source build that is known to be unsupported on the target implementation.

## What the repository actually exposes

In the original PEP 658 specification, a distribution's Simple API anchor can contain:

```text id="2w4c7m"
data-dist-info-metadata
```

The presence of the attribute indicates that the repository can provide the distribution's Core Metadata separately. The attribute can also contain a hash for verifying the metadata.

The separately served metadata is associated with the distribution itself rather than being arbitrary project-level information.

This is important for release-level research because different releases and artifacts can have different metadata.

## Metadata availability is optional

PEP 658 does **not** require every repository to provide separately served metadata.

If the attribute is absent, clients are expected to fall back to their existing behavior, which may involve downloading the distribution to inspect its metadata.

Therefore:

```text id="n8c3vp"
Core Metadata field exists
        !=
metadata is always available before artifact download
```

and:

```text id="q5h7dx"
PEP 658 is supported somewhere
        !=
every repository/client workflow can use it
```

This limitation is critical when evaluating claims that a new metadata field would prevent unnecessary downloads or builds.

## PEP 658 does not add new metadata semantics

PEP 658 is a **transport mechanism**.

It serves the distribution's existing Core Metadata; it does not define a new meaning for individual Core Metadata fields. The metadata served separately must be identical to the distribution's canonical metadata.

Therefore the research should distinguish:

```text id="k7m2qx"
metadata semantics
```

from:

```text id="d4w8zs"
metadata transport
```

A hypothetical:

```text id="v6j1ra"
Supported-Implementation
```

would require a Core Metadata specification defining its semantics.

PEP 658 would then provide one possible mechanism by which repositories could expose that field before downloading the corresponding artifact.

## Why this matters for the proposal

The potential workflow is:

```text id="e5q2mw"
candidate release
      ↓
PEP 658 Core Metadata
      ↓
Supported-Implementation
      ↓
target implementation comparison
      ↓
skip unsupported release
```

This is potentially useful if the support declaration represents a fact that:

1. is stable for the release;
2. cannot be derived from existing metadata;
3. is useful to a resolver before installation/build;
4. can be obtained from the repository early enough;
5. is trustworthy enough to affect candidate selection.

Without all of those properties, adding the field may provide little practical value.

## The metadata availability test

A particularly important distinction is:

```text id="v0q8hy"
metadata exists
```

versus:

```text id="p3x7mw"
metadata is available early enough
```

Suppose a release has:

```text id="w6j2ka"
Supported-Implementation: cpython
```

but the repository does not expose the Core Metadata separately.

A client may then need to:

```text id="u8k1pv"
download sdist
    ↓
inspect metadata
    ↓
discover unsupported implementation
```

The proposed field would still communicate useful information, but it would not necessarily solve the pre-download or pre-build problem that motivated it.

Therefore any claimed consumer benefit must specify the metadata-access path.

## PEP 658 and source-build avoidance

The strongest potential application is an sdist candidate:

```text id="r7m4qn"
candidate sdist
      ↓
separately served Core Metadata
      ↓
implementation support check
      ↓
avoid build if unsupported
```

This could avoid an expensive build attempt.

But the research must not assume that this is automatically superior to existing mechanisms.

For each source-build case, we still need to examine:

```text id="j9c4xv"
Requires-Python
PEP 508 dependency markers
build-system requirements
host requirements
external dependency metadata
wheel tags
build configuration
failed-build caching
```

If another mechanism already provides the relevant decision, the case should not count as evidence for new support metadata.

## Relationship to PEP 643

PEP 643 establishes the semantics needed for reliable Core Metadata in source distributions.

PEP 658 provides a repository-level mechanism for serving that metadata separately.

Together they create a useful architectural chain:

```text id="h2r6vp"
PEP 643
    ↓
reliable Core Metadata in sdist
    ↓
PEP 658
    ↓
metadata can be served separately
    ↓
consumer can inspect metadata before
downloading the full artifact, when available
```

This makes the proposed workflow technically plausible.

Neither PEP establishes that implementation support belongs in Core Metadata.

## Relationship to PEP 714

PEP 714 subsequently renamed the PEP 658 Simple API metadata attributes.

For HTML:

```text id="c4n8qa"
data-core-metadata
```

For JSON:

```text id="m1v7zs"
core-metadata
```

PEP 714 explicitly describes itself as renaming the metadata provided by PEP 658 in both HTML and JSON representations; the underlying supported values remain the same.

Therefore current research should use the PEP 714 names rather than the original PEP 658 names when discussing the current Simple API.

## Current limitation

PEP 658 does not make metadata universally available.

If a repository does not provide the separate metadata representation, clients must retain the previous behavior. Older clients also ignore the new metadata mechanism and continue operating as before.

Therefore a proposed implementation-support field cannot honestly be advertised as:

```text id="z6f4jw"
always visible to resolvers before download/build
```

unless the repository and client ecosystem provide that path.

The field would improve the information available **where Core Metadata is available**, not universally.

## Consumer-decision test

For a proposed implementation-support field, the relevant question is therefore not merely:

```text id="r8v2kc"
Can the field be transported before installation?
```

It is:

```text id="a5q9mn"
Does early access to this field enable a
useful consumer decision that existing mechanisms
cannot make?
```

The evidence should demonstrate:

```text id="f3w7xp"
1. A genuine implementation-specific release boundary.

2. Existing metadata is insufficient.

3. The information is stable at release level.

4. The metadata can be obtained before the costly operation
   the proposal seeks to avoid.

5. The consumer can act on it meaningfully.

6. The benefit remains after considering repository support
   and client compatibility.
```

## Current conclusion

PEP 658 establishes strong **metadata-transport prior art**.

It makes it possible for repositories to expose Core Metadata separately from the associated wheel or sdist, allowing clients to inspect metadata without downloading the complete artifact when the repository provides the facility.

This makes a potential workflow such as:

```text id="y3n6qc"
index
  ↓
Core Metadata
  ↓
implementation-support decision
  ↓
download/build only if appropriate
```

technically plausible.

However:

```text id="e7q1ms"
metadata transport
    !=
metadata semantics
```

and:

```text id="s4k8vw"
metadata availability
    !=
universal early visibility
```

PEP 658 therefore supports the **feasibility** of an early release-metadata decision but does not establish the **necessity** of a new implementation-support field.

**Current conclusion:** PEP 658 is enabling transport infrastructure, not implementation-support prior art. The research must still prove that implementation support is a missing release-level semantic and that early access to that semantic produces a concrete consumer benefit.
