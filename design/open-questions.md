# Open Questions

This document deliberately records questions that the research has not
resolved.

## 1. What does "supported" mean?

A field such as:

```toml
supported-implementation = ["cpython"]
```

could mean several things:

- tested by CI;
- officially supported by maintainers;
- expected to work;
- known to work;
- supported for installation;
- supported for runtime execution;
- supported for building;
- supported for all configurations of that implementation.

These meanings are not equivalent.

A standard must define the intended semantics precisely.

## 2. Is the field normative or informational?

Possible models:

### Informational

The field describes maintainer intent but does not affect installation.

### Advisory

Tools may warn when the current implementation is not listed.

### Candidate-selection metadata

Resolvers/installers may use the field to eliminate candidates.

### Mandatory compatibility constraint

Installers must reject a candidate when the current implementation is not
listed.

The research currently does not select one of these.

## 3. What does absence mean?

Possible choices:

A. absence means all implementations are supported;
B. absence means unknown;
C. absence means no compatibility declaration.

The research currently prefers C.

This is important for backwards compatibility.

Existing distributions must not suddenly become incompatible simply because
they predate the field.

## 4. What does an empty list mean?

Possible meanings:

```toml
supported-implementation = []
```

could mean:

- no implementations supported;
- no information;
- invalid metadata.

The safest design may be to prohibit an empty list.

## 5. Is implementation identity sufficient?

No.

A project can support:

```text
CPython
```

but reject:

```text
CPython free-threaded
```

or:

```text
CPython debug
```

PEP 780 is relevant here.

The proposed field should therefore not become a general-purpose interpreter
compatibility language.

## 6. How should ABI metadata interact with implementation metadata?

A future candidate-selection algorithm could conceptually evaluate:

```text
implementation identity
+
Python version
+
ABI features
+
platform
+
wheel tags
```

The standards need to define which mechanism owns each dimension.

## 7. Runtime vs build-time compatibility

Consider two statements:

```text
The package can only run on CPython.
```

and:

```text
The package's build process must execute under CPython.
```

They may require different metadata semantics.

PEP 725 should be considered before creating a second build-environment
dependency vocabulary.

## 8. Could PEP 725 solve the problem?

PEP 725 is relevant to external dependencies and build/host requirements.

The current PEP does not define Python implementations themselves as the
virtual dependency vocabulary needed for this use case.

Possible future work could investigate whether Python implementations should
be modeled there.

That possibility should be evaluated before claiming a new metadata field is
necessary.

## 9. Could Trove classifiers solve the problem?

Classifiers already allow projects to say:

```text
Programming Language :: Python :: Implementation :: CPython
```

However, classifiers are descriptive classification data.

The unresolved question is whether the packaging ecosystem needs the same
information with normative semantics and defined installer behavior.

## 10. Could wheel tags solve the problem?

For an already-built wheel, wheel tags are the established mechanism.

The difficult case is:

```text
sdist only
```

where the compatible wheel does not yet exist and the installer may need to
build the source.

The research therefore treats wheel tags and release-level metadata as
different layers rather than competing replacements.

## 11. Does Core Metadata need to be extended?

Core Metadata is a plausible place for release-level compatibility data.

But adding a field has costs:

- specification complexity;
- build-backend support;
- metadata validation;
- installer behavior;
- repository behavior;
- documentation;
- backwards compatibility;
- maintenance of the vocabulary.

The benefit must justify those costs.

## 12. How would metadata be obtained?

PEP 658 and PEP 714 allow repositories to expose Core Metadata separately.

However, metadata sidecars are optional.

Therefore a design cannot assume that every package index will provide the
field before an artifact is downloaded.

## 13. Should metadata be release-level or artifact-level?

This research currently prefers release-level semantics.

A release can contain:

- an sdist;
- multiple wheels;
- wheels for different platforms;
- wheels for different ABIs.

Wheel tags already describe the individual wheel artifacts.

The proposed field would instead describe the compatibility claim of the
release.

## 14. What happens when a project publishes an incorrect declaration?

Possible approaches include:

- informational semantics;
- warning only;
- installer rejection;
- validation tooling;
- automated CI verification.

The answer affects whether the field is trustworthy enough for automated
candidate selection.

## 15. Can the implementation vocabulary remain open?

PEP 421 deliberately uses an implementation identity rather than a closed
registry.

A standard should avoid creating a second incompatible registry if possible.

## 16. Does the problem justify a new standard?

This remains the central research question.

The strongest case currently is:

> Some releases have implementation-specific compatibility requirements,
> and an sdist does not expose those requirements through a dedicated
> normative compatibility field before its build is attempted.

The research still needs to establish whether that problem is common and
important enough to justify a new standard mechanism.
