# Sdist Build Avoidance

## Problem

An index can provide:

```text
project-version
├── compatible wheels
└── sdist
```

When no compatible wheel is selected, an installer may attempt the sdist.

The build can fail because of:

- unsupported implementation;
- unsupported platform;
- missing compiler;
- missing external dependency;
- incompatible ABI;
- project policy.

## Why it matters

The ecosystem has a broader need for pre-build information.

But:

```text
"Do not automatically build this sdist"
```

is not equivalent to:

```text
"This release does not support PyPy"
```

A source-build policy mechanism could solve the first problem without
communicating the second.

## Important residual test

For a package with:

```text
py3-none-any wheel
CPython-only producer policy
```

source-build avoidance alone does not explain why the existing wheel should be
rejected.

Therefore the two problems should remain separate in the research.

Current conclusion:

> Sdist-build avoidance is adjacent prior art and a possible alternative for
> some cases, but not a complete semantic replacement.
