# Research Status — 2026-09-07

## Current conclusion

The research has demonstrated a real semantic distinction between:

```text
artifact compatibility
```

and:

```text
producer-declared implementation support
```

The strongest evidence comes from releases that publish implementation-generic
Python wheel tags while explicitly declaring CPython-only support.

Examples currently include:

- RestrictedPython 8.5
- HAX 0.3.0
- Likepy 0.3.0
- simple-ctx-log 0.0.3
- TribeCore 4.7.3

## What is not established

The research has **not** established that a new Core Metadata field is
necessary.

In particular, these alternatives remain viable:

- continued use of classifiers;
- stronger semantics attached to a new or existing classifier;
- index-level metadata;
- source-build policy metadata;
- better resolver heuristics;
- no new standard.

## Current strongest hypothesis

If a new mechanism is justified, a positive producer declaration such as:

```text
Supported-Implementation: cpython
```

appears semantically safer than:

```text
Requires-Implementation: cpython
```

because the empirical cases are primarily support-policy statements.

This is still a hypothesis.

## Next work

The next repository iteration should focus on:

1. exact resolver behavior;
2. consumer demand;
3. Core Metadata vs index-level metadata;
4. whether classifiers can safely carry the required semantics;
5. a reproducible release-level corpus.
