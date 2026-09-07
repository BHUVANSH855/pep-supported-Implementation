# PEP 825 Notes — Wheel Variants

**URL:** https://peps.python.org/pep-0825/
**Status:** Draft / active packaging proposal
**Relevance:** High — important adjacent artifact-compatibility mechanism

## Purpose

PEP 825 defines a mechanism for publishing and selecting variant wheels when
ordinary wheel tags are not sufficient.

It is important because it is a plausible alternative answer to the question:

> Could implementation support simply become another wheel variant property?

## What PEP 825 addresses

PEP 825 adds variant metadata to wheels and an index-level:

```text
{name}-{version}-variants.json
```

The index-level metadata merges the variants available for a project version.

The PEP describes variant properties such as hardware/software compatibility
for particular wheel builds.

Source:
https://peps.python.org/pep-0825/

## Important boundary

PEP 825 is about **artifacts / wheel variants**.

The proposed research question is about **release-level producer support**.

Compare:

```text
PEP 825:
    Which wheel variant should I install?

Research question:
    Does this release support this Python implementation?
```

These can be related without being identical.

## Residual-case test

Consider:

```text
foo-1.0-py3-none-any.whl
foo-1.0.tar.gz
```

with project documentation saying:

```text
CPython only
```

Changing the wheel into a CPython-specific variant would potentially make
the artifact description less accurate if the wheel has no implementation
specificity at the binary/artifact level.

The producer's restriction is semantic support policy.

## Current conclusion

PEP 825 is complementary.

The research should not claim:

> PEP 825 cannot ever represent implementation properties.

Instead:

> Even if implementation properties can be represented for a wheel variant,
> that does not automatically provide a release-level declaration for sdists
> or for implementation-generic artifacts.

That distinction needs to be tested against actual resolver designs.
