# PEP 425 Notes — Wheel Compatibility Tags

**URL:** https://peps.python.org/pep-0425/

**Status:** Final

**Relevance:** High

## What PEP 425 does

PEP 425 defines a compatibility-tagging system for built distributions.

A wheel compatibility tag has three dimensions:

```text id="7l5q3a"
python tag
abi tag
platform tag
```

The complete form is:

```text id="1cbx0y"
{python tag}-{abi tag}-{platform tag}
```

For example:

```text id="q4j8z2"
py3-none-any
cp312-cp312-manylinux_2_17_x86_64
```

PEP 425's purpose is to provide enough information for an installer to determine whether a particular built distribution is compatible with the target environment without first reading the distribution's complete metadata.

## Python tags and implementation identity

The Python tag is explicitly defined as identifying the implementation and Python version required by the distribution.

PEP 425 gives the initial implementation abbreviations:

```text id="b2c7yx"
py  -> Generic Python
cp  -> CPython
ip  -> IronPython
pp  -> PyPy
jy  -> Jython
```

It also states that other Python implementations should use `sys.implementation.name`.

The current platform compatibility-tag specification retains this model and explicitly references `sys.implementation.name` for other implementations.

This establishes an important piece of prior art:

```text id="9r7gq1"
Python implementation identity
        ↓
already participates in
        ↓
built-artifact compatibility selection
```

## What the tags allow an installer to decide

PEP 425 describes the installer as maintaining a set of supported compatibility tags and comparing those tags with the tags of available built distributions.

If a distribution's tag is compatible with the installer's supported tags, that artifact can be selected.

This is an intentionally early filtering mechanism:

```text id="9q0n7b"
Target environment
        ↓
supported compatibility tags
        ↓
candidate wheel tags
        ↓
compatible artifact candidates
```

The mechanism is therefore not merely descriptive metadata. Wheel tags participate directly in artifact selection.

## Generic Python tags

The `py` tag is especially important to this research.

PEP 425 defines:

```text id="m2m8kx"
py = Generic Python
```

and describes it as not requiring implementation-specific features.

For example:

```text id="9s6p4k"
py3-none-any
```

represents a pure-Python artifact intended to be compatible across Python 3 implementations, subject to the semantics of the tag and the other compatibility constraints.

This creates the key empirical question for the present research:

```text id="5jv1r3"
Can a release legitimately publish a generic artifact
while its producer support policy is narrower than
the artifact's compatibility tag?
```

The answer cannot simply be inferred from the filename.

It must be established from the release's actual behavior, declared support policy, and root cause.

## Wheel compatibility is not the same as release support

The most important distinction for this research is:

```text id="v3q4t7"
wheel compatibility
        !=
release-level support policy
```

A wheel tag describes the compatibility of a **particular built artifact**.

A hypothetical Core Metadata field would describe a property of the **distribution release**.

Conceptually:

```text id="9j8b0w"
Wheel tag:

    Can this particular wheel artifact
    be installed for this environment?

Support declaration:

    Does the producer claim that this
    distribution release supports this implementation?
```

These questions can have different answers.

## The importance of the PEP 425 METADATA boundary

PEP 425 explicitly explains why compatibility tags are not placed in the ordinary `METADATA` / `PKG-INFO` fields:

> “METADATA / PKG-INFO should be valid for an entire distribution, not a single build of that distribution.”

This is highly relevant prior art.

It demonstrates that the packaging ecosystem already distinguishes:

```text id="h9c0na"
distribution-wide metadata
```

from:

```text id="5f1p0c"
artifact-specific compatibility information
```

That distinction should be preserved rather than blurred by treating wheel tags as a general-purpose substitute for every possible release-level compatibility declaration.

## The strongest residual case

A potential residual case therefore looks like:

```text id="0t2q6c"
Distribution release:
    X

Artifact:
    py3-none-any

Producer/runtime evidence:
    release does not support implementation Y

Root cause:
    implementation-specific runtime restriction

Existing wheel mechanism:
    accurately describes the artifact

Remaining question:
    how can an installer or index know the
    release-level support boundary before
    installing/building/testing it?
```

This is substantially stronger than saying:

```text id="r7w2mv"
The wheel tag is wrong.
```

The research should not assume the tag is wrong.

The artifact may genuinely be generic.

The unresolved question, if one exists, is whether **producer support policy is an additional release-level fact** that cannot be represented by the artifact tag.

## `py3-none-any` is not automatically evidence of universal support

The repository's empirical examples include releases such as:

* RestrictedPython 8.5;
* HAX 0.3.0;
* Likepy 0.3.0;
* simple-ctx-log 0.0.3.

Some of these publish generic-looking pure-Python wheels while making narrower implementation-support claims.

These cases should be treated as empirical candidates, not as automatic proof that wheel tags are inadequate.

For each case, research must still establish:

1. the exact release;
2. the exact artifact tag;
3. the producer's actual support statement;
4. the technical or policy reason for the implementation boundary;
5. whether the generic artifact is genuinely installable;
6. whether existing metadata already provides an equivalent constraint;
7. what concrete consumer decision would become possible from an additional release-level declaration.

## Artifact-level implementation restriction is already supported

PEP 425 demonstrates that implementation-specific artifact restrictions are already expressible.

For example:

```text id="u6q1z8"
cp312-cp312-...
```

can communicate that an artifact is specific to CPython 3.12 and its corresponding ABI.

Similarly:

```text id="3z4m1k"
pp3-...
```

can represent an implementation-specific PyPy artifact where the applicable tag is defined.

The historical packaging problem addressed by PEP 425 was precisely that artifacts built for different implementations could otherwise share indistinguishable filenames even when they were incompatible.

Therefore the research must explicitly exclude cases where the desired restriction is already an **artifact compatibility** restriction.

## Multiple artifacts for one release

PEP 425 also allows multiple compatible built distributions for a package release.

For example, a project may provide:

```text id="k3z7qa"
cp...   -> optimized/native implementation
py3...  -> pure-Python fallback
```

The installer can prefer the more specific artifact and fall back to the generic one when appropriate.

This is important evidence against treating every implementation-specific behavior as requiring new release metadata.

A project may legitimately support multiple implementations through different artifacts.

The residual case is strongest only when:

```text id="j8w4q2"
the release itself has an implementation-support boundary
```

that remains invisible to existing artifact and metadata mechanisms.

## PEP 425 and source distributions

Wheel tags describe built distributions.

They do not directly provide an equivalent implementation-compatibility declaration for an sdist.

This matters because one proposed use case for implementation support metadata is preventing an installer from selecting an sdist that cannot successfully build on the target implementation.

However, this is not automatically a justification for new metadata.

The research must separately consider:

```text id="f2s9vb"
- source-build behavior;
- build-system requirements;
- host requirements;
- dependency constraints;
- build failure caching;
- existing build metadata;
- implementation-specific build logic.
```

In particular, a source-build failure caused by an external dependency or build/host requirement may belong to existing or newer packaging mechanisms rather than a general implementation-support field.

## `py` does not mean "all implementations"

The semantics of the generic Python tag should be interpreted carefully.

PEP 425 defines `py` as generic Python and says that it does not require implementation-specific features.

That is an artifact compatibility statement.

It should not automatically be transformed into the broader proposition:

```text id="x8v3a1"
The producer promises support for every
Python implementation.
```

This distinction is central to the research.

An artifact may contain no implementation-specific code while the project still has a narrower support policy, for example because the producer has only validated a particular implementation or because a runtime-level assumption is outside the artifact's compatibility-tag semantics.

Whether such a policy should be machine-readable is the unresolved question.

## Current conclusion

Do **not** claim that wheel compatibility tags are inadequate.

PEP 425 successfully solves an important and well-defined problem:

> determining compatibility of a particular built distribution with a target environment before downloading or installing it.

The current research should instead preserve the following distinction:

```text id="8w2r5c"
Wheel compatibility tags
    ->
    artifact-level compatibility

Core Metadata
    ->
    distribution/release-level metadata

Possible future support declaration
    ->
    producer-declared release-level
    implementation support
```

The existence of this semantic distinction does **not** by itself justify a new field.

The strongest remaining question is narrower:

> Can a distribution release have a genuine implementation-support boundary that cannot honestly be represented by its existing wheel tags, `Requires-Python`, dependency metadata, build metadata, or other existing mechanisms, and for which an installer or index could make a useful pre-install or pre-build decision?

Only residual cases satisfying that test should count as evidence for a new Core Metadata field.

PEP 425 is therefore both **existing-mechanism evidence** and an important **boundary control**: implementation-specific artifact compatibility is already standardized and should not be duplicated by a release-level support field.
