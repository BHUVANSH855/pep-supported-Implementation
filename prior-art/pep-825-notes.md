# PEP 825 Notes — Wheel Variants

**URL:** https://peps.python.org/pep-0825/

**Status:** Draft — proposal under active discussion and revision

**Relevance:** High — important adjacent artifact-compatibility mechanism

## What PEP 825 addresses

PEP 825 defines a format for **variant wheels**.

Its purpose is to allow multiple wheels for the same package version to express additional compatibility properties when ordinary wheel platform compatibility tags are insufficient.

The motivating examples include hardware and software compatibility dimensions such as:

```text id="4x8m2p"
GPU support
CPU instruction sets
CUDA versions
BLAS/LAPACK implementations
other hardware/software properties
```

The PEP is explicitly concerned with selecting the most appropriate **wheel artifact** for a target environment. ([peps.python.org](https://peps.python.org/pep-0825/))

## Variant properties

PEP 825 introduces structured variant properties represented conceptually as:

```text id="6m4q7v"
namespace :: feature :: value
```

A variant wheel stores its variant metadata in:

```text id="p7x2cn"
*.dist-info/variant.json
```

and the wheel filename contains a variant label associated with those properties. ([peps.python.org](https://peps.python.org/pep-0825/))

This provides a much richer compatibility vocabulary than ordinary wheel tags.

## Index-level variant metadata

When variant wheels are hosted on an index, PEP 825 defines:

```text id="w5n8kc"
{name}-{version}-variants.json
```

as index-level metadata describing the variants available for that package version.

The purpose is specifically to avoid fetching multiple variant wheels merely to discover their variant properties during dependency resolution. ([peps.python.org](https://peps.python.org/pep-0825/))

This is important prior art for early candidate selection:

```text id="q2m6vx"
index
    ↓
variant metadata
    ↓
candidate compatibility
    ↓
select appropriate wheel
```

## Why PEP 825 matters here

PEP 825 is a serious alternative mechanism whenever the observed problem is actually:

```text id="n4c8wy"
Which built artifact should be installed?
```

rather than:

```text id="h7m3qp"
Does the release itself claim support for
this Python implementation?
```

That distinction must be tested empirically.

A proposed implementation-support field should not duplicate an artifact-selection mechanism that already exists or is being standardized for the relevant compatibility dimension.

## Important boundary

The central distinction remains:

```text id="f6q2mw"
PEP 825:
    Which wheel variant is compatible/preferred?

Research proposal:
    Which Python implementations does this release support?
```

These questions can interact, but they are not identical.

PEP 825 describes compatibility properties of **wheel variants**.

A hypothetical Core Metadata field would describe a **release-level producer declaration**.

## Variant properties are artifact/build properties

PEP 825 states that variant properties express compatibility of binary packages with specific platforms in addition to ordinary platform compatibility tags. ([peps.python.org](https://peps.python.org/pep-0825/))

This means variant metadata is naturally associated with:

```text id="j3w8vn"
a particular built package
```

rather than automatically with every artifact of the release.

That distinction is especially important for source distributions.

An sdist does not itself select a GPU-specific or CPU-specific binary variant.

Therefore:

```text id="c5q7mx"
variant wheel metadata
    !=
general release support declaration
```

## PEP 825 explicitly keeps variant metadata separate from Core Metadata

A particularly important architectural detail is that PEP 825 stores variant metadata in its own JSON format rather than adding those properties to Core Metadata.

The PEP explains that this metadata is versioned independently and can be ignored by tools that do not need to understand variant compatibility. ([peps.python.org](https://peps.python.org/pep-0825/))

This is useful negative prior art.

It demonstrates that the packaging ecosystem does not automatically put every new compatibility dimension into Core Metadata.

Instead, the appropriate representation depends on the semantic scope of the information.

## Could implementation identity become a variant property?

In principle, PEP 825's structured variant-property mechanism is flexible enough to represent additional compatibility dimensions.

The PEP explicitly allows independently governed variant namespaces and feature/value combinations. ([peps.python.org](https://peps.python.org/pep-0825/))

Therefore the research should **not** make the absolute claim:

> PEP 825 cannot represent implementation properties.

That is too strong.

The correct question is:

```text id="s8m2kp"
Would implementation support be correctly modeled
as an artifact/variant compatibility property?
```

That requires examining the semantics of the actual use case.

## The release-level problem remains

Suppose a release publishes:

```text id="k4n7xz"
foo-1.0-py3-none-any.whl
foo-1.0.tar.gz
```

and the producer states:

```text id="w8q2mc"
CPython only
```

If the wheel is genuinely implementation-generic at the artifact level, turning it into a CPython-specific variant merely to encode the producer's support policy could change the meaning of the artifact.

The resulting variant would imply an artifact compatibility restriction.

But the producer's original statement may instead mean:

```text id="r6m3vp"
The artifact is technically generic,
but the producer only promises support for CPython.
```

Those are different claims.

The research must determine which claim is actually supported by the evidence.

## Why this distinction matters for sdists

A release-level support declaration could apply to:

```text id="b5q8wn"
sdist
wheel A
wheel B
wheel C
```

as one release property.

A variant mechanism applies to built wheel artifacts.

Therefore a variant-only solution would not automatically communicate:

```text id="y7m2kc"
this source distribution release is unsupported
on PyPy
```

before a source build.

This is one of the strongest boundaries between the two mechanisms.

## Variant selection does not equal support policy

PEP 825's resolver flow is approximately:

```text id="n2v6jq"
package version
    ↓
available wheels
    ↓
ordinary wheel compatibility
    ↓
variant compatibility
    ↓
variant ordering
    ↓
select best wheel
```

That is a candidate-selection problem.

A release-support declaration would instead potentially provide:

```text id="z4c8mx"
package release
    ↓
producer support declaration
    ↓
is this implementation supported?
```

The second statement could influence candidate selection, but it is not itself a variant-selection rule.

## PEP 825 and generic wheels

A particularly important residual case is:

```text id="m9x3qw"
generic wheel
    +
narrow producer support policy
```

If the wheel contains no implementation-specific binary compatibility requirement, representing the policy as an artifact variant may be semantically questionable.

The research should therefore distinguish:

```text id="v8q2ny"
artifact requires implementation X
```

from:

```text id="d5m7kc"
producer supports implementation X only
```

The former belongs naturally to compatibility metadata.

The latter is the proposed residual category.

## PEP 825 and multiple artifacts

PEP 825 is particularly strong evidence against using release-level metadata to describe something that genuinely varies between wheel builds.

For example:

```text id="j6q4wp"
Release 2.13.0

    wheel A -> CUDA 12.6
    wheel B -> CUDA 13.0
    wheel C -> ROCm 7.2
    wheel D -> CPU fallback
```

Here the differences are intentionally artifact/variant-specific.

A release-level field saying:

```text id="r3n8vy"
Supported-Implementation: ...
```

would not replace the need for variant selection.

This is an important scope boundary.

## PEP 825 and source distributions

PEP 825 does not solve the general question:

```text id="x4m7qp"
Can this source distribution release be built
and supported on implementation X?
```

Its variant-selection mechanism operates on wheels.

Therefore, if the motivating use case is specifically:

```text id="w6c2zn"
avoid building an unsupported sdist
```

PEP 825 is not by itself a complete answer.

However, the research must still investigate whether the source-build restriction is better represented by:

```text id="k8q3mv"
build/host requirements
```

or other existing mechanisms.

## Relationship to PEP 425

PEP 825 should be understood as an extension of the broader artifact-compatibility architecture rather than a replacement for it.

Conceptually:

```text id="f9m2kc"
PEP 425
    ->
standard wheel compatibility tags

PEP 825
    ->
additional wheel variant compatibility properties
```

Both concern selection of a built artifact.

This reinforces the boundary:

```text id="q5n8vx"
artifact compatibility
    !=
release-level support policy
```

## Relationship to PEP 780

PEP 780 and PEP 825 also operate at different levels.

PEP 780 concerns environment/ABI features.

PEP 825 provides a mechanism for wheels to declare additional compatibility properties and for tools to select among variants.

Therefore an implementation-support proposal should not absorb ABI or hardware features simply because variant wheels can represent them.

The mechanisms should remain layered according to their semantic scope.

## Residual-case test

For a candidate case to remain evidence for release-level implementation-support metadata after considering PEP 825, establish:

```text id="a7q4nw"
1. The issue is not merely selecting among wheel artifacts.

2. Existing wheel tags are insufficient.

3. Variant properties do not accurately describe the
   producer's actual support statement.

4. The support boundary applies to the release,
   not merely to one wheel build.

5. The same issue matters for sdists or otherwise
   cannot be reduced to wheel selection.

6. Existing build/host/dependency mechanisms are insufficient.

7. A consumer would make a materially better decision
   from release-level support information.
```

Only then should the case remain a residual candidate.

## Current conclusion

PEP 825 is important **adjacent artifact-compatibility prior art**.

It demonstrates that the packaging ecosystem is actively developing mechanisms for compatibility dimensions that ordinary wheel tags cannot represent, including index-level metadata that can support efficient variant selection. ([peps.python.org](https://peps.python.org/pep-0825/))

It therefore strengthens the requirement that the research distinguish:

```text id="h2m7qx"
artifact compatibility
```

from:

```text id="p9c4wv"
release-level producer support
```

The research should **not** claim:

> PEP 825 cannot represent implementation properties.

Instead:

> Even if implementation properties can be represented as wheel variant properties, that does not automatically provide a release-level declaration applying to sdists and implementation-generic artifacts.

The strongest remaining question is therefore:

```text id="r6w3kn"
Is the observed implementation restriction
actually an artifact compatibility property,
or is it a release-level producer support policy?
```

If it is the former, PEP 425/825-style mechanisms may be sufficient.

If it is the latter, the research must still establish whether that release-level fact is sufficiently valuable and otherwise unrepresented to justify new Core Metadata.
