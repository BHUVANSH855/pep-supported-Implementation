# PEP 621 Notes

**URL:** https://peps.python.org/pep-0621/

**Relevance:** Medium

## What PEP 621 does

PEP 621 standardizes how projects declare core metadata in:

```toml
[project]
```

The corresponding current packaging specification describes `[project]` as the table for declaring a project's core metadata. Metadata may be specified statically or, where permitted, declared as `dynamic` so that a build backend provides it later.

Examples include:

```toml
[project]
name = "example"
version = "1.0"
requires-python = ">=3.10"
classifiers = [
    "Programming Language :: Python :: Implementation :: CPython",
]
```

The important architectural point for this research is that `[project]` is an **authoring representation** of standardized project metadata.

## PEP 621 does not itself define new Core Metadata

PEP 621 explicitly states that it is not changing the underlying Core Metadata specification.

Its purpose is to provide a standardized way to specify existing core metadata in `pyproject.toml`. Changes or additions to the underlying Core Metadata are to be handled separately.

This distinction matters for a hypothetical implementation-support field.

The research should not reason:

```text
A new [project] key
        ->
automatically a new Core Metadata field
```

Instead, the standardization path would conceptually be:

```text
new semantic requirement
        ↓
Core Metadata specification change
        ↓
project-metadata authoring specification
        ↓
[project] representation
        ↓
build-backend serialization
```

The exact process and naming would depend on the eventual standards proposal.

## Hypothetical project declaration

If a release-level support field were eventually justified, a project-level representation might look conceptually like:

```toml
[project]
supported-implementations = ["cpython"]
```

This is only an illustrative design at this stage.

It must **not** be treated as an established PEP 621 field.

A real proposal would need to define at least:

```text
field name
value type
value vocabulary
Core Metadata mapping
validation rules
static/dynamic behavior
release consistency
interaction with classifiers
interaction with Requires-Python
```

and any additional resolver or build-system semantics.

## Static versus dynamic metadata

PEP 621 provides an important precedent for deciding whether a proposed field should be statically declared or generated dynamically.

The `dynamic` mechanism explicitly distinguishes metadata that the project author has intentionally left for a build backend to provide from metadata that was statically specified. Build backends must honor statically specified metadata and may only provide metadata dynamically when the project has opted into that behavior.

This matters because implementation support is potentially a property of a **specific release**.

For example, if a project declares:

```toml
[project]
supported-implementations = ["cpython"]
```

the resulting distribution metadata would need to preserve a clear relationship between:

```text
project declaration
        ↓
built distribution
        ↓
released metadata
```

A build backend should not silently replace a producer's declared support boundary with its own interpretation.

Conversely, if support information is inherently generated from source inspection or build configuration, the standard would need to decide whether dynamic generation is appropriate.

These are design questions for a future proposal, not semantics supplied by PEP 621 itself.

## Release-level consistency

PEP 621 also highlights an important distinction between **project metadata authoring** and the final metadata attached to a particular distribution.

A project declaration may be consumed by a build backend to produce release artifacts, but the resulting Core Metadata is what packaging consumers ultimately receive.

For implementation-support metadata, the research should therefore ask:

```text
Is support declared once for the project?

or:

Is support potentially different for each release?
```

The second model is likely more relevant to this research because the proposed field is intended to describe a particular released distribution.

The evidence corpus already contains examples where implementation support can change between releases.

Therefore the metadata must not accidentally encode an immutable project-wide property if the intended semantics are release-specific.

## Interaction with classifiers

PEP 621 maps:

```toml
[project]
classifiers = [...]
```

to the Core Metadata `Classifier` field.

This provides an existing authoring mechanism for implementation-related descriptive information, for example:

```text
Programming Language :: Python :: Implementation :: CPython
```

However, classifiers have different semantics from a hypothetical normative support field.

The research should therefore distinguish:

```text
Classifier
    -> descriptive/classification metadata

Possible support field
    -> normative release-level support declaration
```

A new field should not simply duplicate the meaning of an existing classifier unless there is a clearly demonstrated difference in consumer semantics.

Conversely, the existence of classifiers does not automatically make a normative field unnecessary if the corpus establishes a real need for machine-actionable support information.

## Interaction with `Requires-Python`

PEP 621 already provides:

```toml
[project]
requires-python = "..."
```

which maps to:

```text
Requires-Python
```

This is another example of a project-level declaration that becomes standardized Core Metadata.

Conceptually:

```text
requires-python
    -> Python language-version requirement

hypothetical supported-implementations
    -> Python implementation support declaration
```

The similarity is useful for authoring-model design, but it does not establish that the second field is necessary.

The research must first demonstrate that implementation support cannot be represented adequately through existing metadata and artifact mechanisms.

## Interaction with PEP 508 dependencies

PEP 621's `dependencies` field accepts PEP 508 dependency strings and maps them to `Requires-Dist`.

Therefore a project can already express implementation-conditional dependencies from the `[project]` table:

```toml
[project]
dependencies = [
    'cffi; implementation_name == "pypy"',
]
```

This reinforces the distinction established in the PEP 508 research:

```text
conditional dependency
    !=
release-level implementation support
```

A new `[project]` key should not be introduced merely because implementation names occur in dependency declarations.

It would require a separate semantic justification.

## The canonicality requirement

PEP 621's treatment of statically specified metadata is especially relevant to a future support declaration.

When metadata is specified statically, build backends must honor it. If a field is listed in `dynamic`, the project has explicitly allowed a tool to provide it later.

For a normative support declaration, this raises an important question:

```text
Who is authoritative?

Project author
    or
build backend
    or
build environment
    or
some combination?
```

A field intended to communicate producer support policy should not accidentally become an implicit statement about the capabilities of the machine that happened to build the release.

For example:

```text
build performed on CPython
```

does not necessarily imply:

```text
release supports only CPython
```

Nor does:

```text
build performed on CPython
```

necessarily establish:

```text
release supports CPython
```

The declaration's source and semantics must therefore be explicit.

## PEP 621 is not evidence of necessity

The existence of a clean `[project]` representation should not be used as an argument that the underlying Core Metadata field is needed.

The logical order must remain:

```text
real-world residual problem
        ↓
consumer decision that matters
        ↓
existing mechanisms insufficient
        ↓
new semantic requirement justified
        ↓
Core Metadata design
        ↓
PEP 621 project-level representation
```

Not:

```text
[project] can represent it
        ↓
therefore we should add it
```

This distinction is particularly important because PEP 621 intentionally does not attempt to standardize every possible kind of project metadata. It focuses on core metadata with sufficiently broad interoperability value.

## What would need to be standardized

If the research eventually supports a new implementation-support field, the authoring layer would need to follow the semantics established at the Core Metadata level.

At minimum, a proposal would need to resolve:

```text
1. Core Metadata field name.

2. [project] key name.

3. Value representation.

4. Implementation identity vocabulary.

5. Exact matching semantics.

6. Whether implementation versions are supported.

7. Whether the field is static, dynamic, or both.

8. Whether omission means "unknown/no declaration".

9. Whether multiple implementations are an allow-list.

10. How the declaration applies to different releases.

11. Interaction with Requires-Python.

12. Interaction with classifiers.

13. Interaction with wheel tags.

14. Interaction with dependency markers.

15. How sdists and wheels preserve the declaration.

16. How tools should handle conflicting or invalid declarations.
```

These are downstream design requirements.

They should not be solved prematurely in the evidence phase.

## Current conclusion

PEP 621 provides the **project-metadata authoring mechanism** that could eventually be used to declare a new Core Metadata field.

It does not itself provide an implementation-support field, and it does not provide evidence that such a field is necessary.

The important architectural distinction is:

```text
PEP 621
    ->
how standardized project metadata is authored

Core Metadata
    ->
what standardized release metadata means

Potential support field
    ->
a new semantic question that must first be justified
```

If the residual-case research ultimately establishes a genuine need for release-level implementation support metadata, PEP 621 provides a plausible place for projects to author that information.

Until then, a hypothetical:

```toml
[project]
supported-implementations = ["cpython"]
```

should be treated only as a **design sketch**, not as a proposed or existing standard.

**Current conclusion:** PEP 621 provides the project-metadata integration layer, including static/dynamic authoring semantics, but it is not evidence that the proposed implementation-support metadata is necessary.
