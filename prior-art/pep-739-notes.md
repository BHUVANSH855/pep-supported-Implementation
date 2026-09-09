# PEP 739 Notes — `build-details.json`

**URL:** https://peps.python.org/pep-0739/

**Relevance:** Medium — useful boundary for environment/build information

## What PEP 739 addresses

PEP 739 defines `build-details.json`, a standardized structured description of a Python installation/build.

It is concerned with information about the Python installation and the way that Python was built, including implementation and build characteristics.

This makes it useful prior art for separating **information about an environment** from **metadata about a distribution release**.

## The important direction of information

PEP 739 primarily describes the Python installation/build environment:

```text id="7w2k9p"
Python installation
        ↓
build-details.json
        ↓
information about that installation/build
```

The research proposal concerns the opposite direction:

```text id="m5q8vx"
distribution release
        ↓
support declaration
        ↓
which Python implementations does the release support?
```

These are different metadata subjects.

## Environment information versus release policy

The distinction can be stated as:

```text id="f3r7qa"
PEP 739:
    What is this Python installation/build?

Research proposal:
    What Python implementation does this
    distribution release support?
```

For example, an environment can report:

```text id="q6n1vz"
sys.implementation.name == "cpython"
```

without establishing:

```text id="t8m4kc"
this release supports CPython.
```

Conversely, a distribution could declare support for CPython without describing all of the build characteristics of the particular CPython installation used to produce an artifact.

## Why this matters

Implementation identity is only one dimension of the environment.

A Python installation can have additional properties relating to:

```text id="z2p7mb"
build configuration
ABI
platform
compiler/toolchain
debug configuration
free-threading
other implementation characteristics
```

PEP 739 therefore belongs to a broader class of environment/build description mechanisms.

A release-support declaration would not replace this information.

Likewise, knowing the complete build description of an interpreter would not automatically tell a resolver what the package producer promises to support.

## Build identity does not imply package support

A particularly important research rule is:

```text id="c8w4sx"
environment in which an artifact was built
        !=
environment on which the release is supported
```

For example:

```text id="n7m3qd"
wheel built using CPython
```

does not necessarily imply:

```text id="a4k8vy"
release supports CPython only.
```

A pure-Python project could be built under CPython and intentionally support PyPy, GraalPy, or other implementations.

Likewise, a project could have a generic wheel while its producer has deliberately established a narrower support policy.

The build environment alone cannot resolve that semantic question.

## Relationship to `sys.implementation`

PEP 421 provides runtime implementation identity through:

```text id="r9c2wj"
sys.implementation.name
```

PEP 739 operates at the structured Python-installation/build-description layer.

These mechanisms describe the **target environment** or Python installation.

A release-support declaration would instead describe the **producer's claim about a distribution**.

Therefore the conceptual layers are:

```text id="v5m1hx"
Python runtime
    ↓
sys.implementation

Python installation/build
    ↓
build-details.json

Distribution release
    ↓
potential support metadata
```

These layers should not be merged.

## PEP 739 is not a replacement for release-support metadata

It would be incorrect to argue:

```text id="e4n7qp"
PEP 739 describes implementation/build details
        ↓
therefore package support can be inferred from it.
```

That inference does not follow.

An environment description can tell a tool what environment it is dealing with.

It cannot necessarily tell the tool whether an unrelated distribution's producer claims support for that environment.

That requires information associated with the distribution itself.

## Conversely, support metadata is not a replacement for PEP 739

The opposite mistake is equally important.

A declaration such as:

```text id="w2q6ka"
Supported-Implementation: cpython
```

would not describe:

```text id="x9m3rv"
whether the target CPython has a particular ABI,
build configuration, compiler, or other environment property.
```

Those are separate compatibility dimensions.

This is especially relevant when considering ABI-sensitive cases.

## Current conclusion

PEP 739 provides useful **environment/build information prior art**.

It reinforces the distinction:

```text id="j7p4ws"
environment identity and build characteristics
        !=
distribution release support policy
```

A resolver can obtain information about the Python installation without thereby knowing which releases claim to support that environment.

Likewise, a release-level implementation-support declaration would not replace detailed environment/build information.

**Current conclusion:** PEP 739 is a scope and layering boundary, not a direct alternative to release-level implementation-support metadata. It strengthens the requirement that the proposed concept describe the **producer's release policy**, rather than merely restating information about the target Python installation.
