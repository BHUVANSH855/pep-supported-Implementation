# Sdist Build Avoidance

## Purpose

There are packaging discussions about avoiding unwanted attempts to build
source distributions.

These discussions provide important problem-space evidence.

## General problem

A package index may contain:

```text
project version
├── wheels for some environments
└── sdist
```

If the resolver cannot find a compatible wheel, it may select the sdist and
attempt to build it.

That build may:

- require native dependencies;
- require a particular build environment;
- fail on unsupported platforms;
- fail on unsupported Python implementations;
- be intended only for redistributors or package maintainers.

## Why this is relevant

The desired outcome in some cases is:

```text
know that the sdist is unsuitable
        ↓
avoid expensive build attempt
```

rather than:

```text
attempt build
        ↓
discover incompatibility
```

## Important limitation

Sdist build avoidance is a broader problem than Python implementation
compatibility.

Possible causes include:

- unsupported platform;
- missing system dependency;
- missing compiler;
- unsupported ABI;
- project policy;
- implementation-specific incompatibility.

Therefore build-avoidance discussions should not be presented as proof that
`Supported-Implementation` is the correct solution.

## Research conclusion

These discussions establish that pre-build candidate information can have
real value.

They do not determine what metadata mechanism should carry that information.
