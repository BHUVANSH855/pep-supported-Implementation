# PEP 621 Notes

**URL:** https://peps.python.org/pep-0621/
**Relevance:** Medium

PEP 621 defines standardized project metadata in:

```toml
[project]
```

A future Core Metadata field would likely need a corresponding project-level
declaration if publishers are expected to author it in `pyproject.toml`.

For example, a hypothetical:

```toml
[project]
supported-implementations = ["cpython"]
```

would require a standards change defining:

- field name;
- value vocabulary;
- Core Metadata mapping;
- validation;
- static/dynamic behavior;
- release consistency;
- interaction with classifiers.

PEP 621 itself does not provide an implementation-support field.

Current conclusion:

> PEP 621 provides the project-metadata integration mechanism, not evidence
> that the proposed field is necessary.
