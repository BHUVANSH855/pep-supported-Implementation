# PEP 421 Notes — `sys.implementation`

**URL:** https://peps.python.org/pep-0421/
**Relevance:** High

PEP 421 standardized `sys.implementation` as a source of interpreter
implementation identity and related details.

## Research relevance

The environment side of the problem is therefore already standardized.

Conceptually:

```text
Environment:
    sys.implementation.name
```

The missing research question is:

```text
Release:
    which implementation names does the producer support?
```

## Important boundary

Do not invent a new environment identity mechanism as part of this proposal.

A future release-support field should, if standardized, define a precise
relationship to `sys.implementation.name`.
