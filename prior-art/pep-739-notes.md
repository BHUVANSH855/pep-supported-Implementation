# PEP 739 Notes — `build-details.json`

**URL:** https://peps.python.org/pep-0739/
**Status:** Accepted / Python 3.14-era standard
**Relevance:** Medium — useful boundary for environment/build information

## What PEP 739 addresses

PEP 739 defines `build-details.json` to describe a Python installation/build
in a structured way.

It is concerned with information about the Python environment/build itself,
including implementation and build details.

## Why it matters here

It demonstrates another layer:

```text
PEP 739:
    What Python installation/build environment is this?

Research proposal:
    Which Python implementations does this release support?
```

Those are inverse directions.

One describes the **environment**.

The other would describe the **producer's release policy**.

## Boundary

A resolver can know:

```text
sys.implementation.name == "cpython"
```

without knowing:

```text
this release supports CPython
```

Likewise a package can declare:

```text
CPython supported
```

without describing every ABI/build property of the target interpreter.

## Current conclusion

PEP 739 strengthens the argument for keeping:

```text
environment identity
```

separate from:

```text
release support
```

It is not a direct alternative to the proposed metadata.
