# PEP 714 Notes

**URL:** https://peps.python.org/pep-0714/
**Status:** Final
**Relevance:** Medium/High

PEP 714 renamed the PEP 658 Simple API metadata attributes to:

```text
data-core-metadata
```

for HTML and:

```text
core-metadata
```

for JSON.

This means a future Core Metadata field can reuse existing repository
metadata transport.

It does not define the semantics of the field.

Current conclusion:

> PEP 714 is transport prior art, not implementation-support prior art.
