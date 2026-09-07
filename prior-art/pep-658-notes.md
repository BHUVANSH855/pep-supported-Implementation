# PEP 658 Notes

**URL:** https://peps.python.org/pep-0658/
**Status:** Final
**Relevance:** High

PEP 658 allows repositories to expose Core Metadata separately from a
distribution file.

A client can therefore potentially inspect metadata without downloading the
entire wheel or sdist.

The mechanism is optional.

## Research relevance

If implementation support were a Core Metadata field, the desired workflow
could be:

```text
index
  ↓
Core Metadata
  ↓
implementation-support decision
  ↓
download/build only if appropriate
```

rather than:

```text
download sdist
  ↓
build
  ↓
discover incompatibility
```

## Important limitation

PEP 658 does not guarantee metadata sidecars.

Therefore the existence of a new field would not mean every resolver can
always see it before downloading an artifact.

Current conclusion:

> PEP 658 makes the proposed workflow technically plausible but does not
> establish that a new field is needed.
