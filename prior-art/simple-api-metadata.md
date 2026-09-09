# Simple API Core Metadata

## Status

**Relevance:** High

The Simple Repository API provides a standardized mechanism for repositories to expose the Core Metadata associated with an individual distribution without requiring an installer to download the entire distribution artifact.

This is important to the proposed implementation-support research because a release-level metadata field is only useful for early candidate selection if consumers can obtain that metadata early enough.

The key distinction is:

```text
Core Metadata semantics
        ≠
metadata transport
```

PEP 658 and PEP 714 address the second problem.

They do not establish what Core Metadata fields should mean.

---

## 1. PEP 658

PEP 658 introduced a mechanism for repositories to expose the Core Metadata file associated with a distribution through the Simple API.

The original mechanism added:

```text
data-dist-info-metadata
```

to distribution links in the HTML Simple API.

When present, the repository provides the distribution's Core Metadata separately, at a predictable `.metadata` URL.

The metadata must be identical to the canonical Core Metadata contained in the distribution.

PEP 658 applies to standards-compliant distributions including:

```text
wheels
sdists
```

The motivation was explicitly to allow package-management tools to inspect distribution metadata without downloading distributions that they ultimately would not install.

Conceptually:

```text
Simple API
    │
    ├── distribution URL
    │
    └── Core Metadata URL
             │
             ↓
        inspect metadata
             │
             ↓
        decide whether
        distribution is needed
```

This is directly relevant to resolver efficiency.

---

## 2. PEP 714

PEP 714 renamed the PEP 658 metadata indicators.

For HTML:

```text
data-core-metadata
```

For the JSON Simple API:

```text
core-metadata
```

PEP 714 was accepted as the normative replacement for the earlier names.

Therefore current research should use:

```text
data-core-metadata
core-metadata
```

rather than treating:

```text
data-dist-info-metadata
```

as the current spelling.

The older name remains relevant as historical prior art and for compatibility with older clients/repositories.

---

## 3. What PEP 658 actually enables

The important workflow is:

```text
index
  ↓
candidate distribution
  ↓
metadata available separately
  ↓
fetch Core Metadata
  ↓
evaluate metadata
  ↓
decide whether candidate is useful
  ↓
download distribution only if necessary
```

For example:

```text
package 1.0
    ├── sdist
    └── wheel

metadata:
    Requires-Python: >=3.10
```

A resolver can inspect the metadata without first downloading the complete artifact.

This is particularly valuable when many candidates need to be examined.

PEP 658 explicitly motivated the mechanism around avoiding unnecessary distribution downloads while resolving dependencies.

---

## 4. Important limitation: metadata availability is optional

The PEP 658 mechanism is optional.

If the Simple API does not provide the metadata indicator, the client cannot assume that a separately served metadata file exists.

PEP 658 specifies that when the attribute is absent, tools are expected to revert to their existing behavior of downloading the distribution to inspect its metadata.

Therefore:

```text
standardized metadata field
        ≠
metadata always available separately
```

and:

```text
PEP 658 support
        ≠
all repositories provide early metadata
```

This is an important constraint on any proposed implementation-support field.

---

## 5. PEP 691 and JSON Simple API

PEP 691 defines a JSON representation of the Simple API.

Its per-file information can include:

```text
requires-python
core metadata availability
hashes
yanked status
```

The JSON representation therefore provides another route through which clients can discover whether Core Metadata is available.

PEP 691 explicitly treats the Core Metadata indicator as optional. If the metadata key is present, the corresponding metadata file is available; if it is absent, the metadata file may or may not exist.

PEP 714 subsequently standardized the current JSON key as:

```text
core-metadata
```

rather than the historical spelling.

---

## 6. Important distinction: `Requires-Python` gets separate Simple API treatment

The Simple API already exposes `Requires-Python` directly at the file-listing level.

PEP 691 defines:

```text
requires-python
```

as an optional per-file value and says installers should ignore the download when installing to a Python version that does not satisfy the requirement.

This creates an important architectural distinction:

```text
Requires-Python
    ↓
can be surfaced directly in the
Simple API candidate listing


Other Core Metadata
    ↓
may require fetching Core Metadata
through PEP 658/714
```

A future implementation-support field would therefore need to consider whether it should be:

```text
only part of Core Metadata
```

or whether an additional Simple API representation would be needed for efficient candidate filtering.

That decision cannot be assumed from PEP 658 alone.

---

## 7. Metadata transport does not define candidate semantics

PEP 658 says how a client can obtain Core Metadata.

It does not say:

> Every Core Metadata field is automatically a candidate-selection constraint.

This is crucial.

For example:

```text
Summary
Description
Classifier
Project-URL
```

can be present in Core Metadata without being hard installation constraints.

By contrast:

```text
Requires-Python
```

has explicit candidate-selection semantics.

Therefore a hypothetical:

```text
Supported-Implementation: CPython
```

would need its own normative specification defining whether an installer:

```text
MUST reject
SHOULD reject
MAY prefer
or merely display
```

a candidate that does not match.

PEP 658 does not answer this question.

---

## 8. Potential resolver workflow for implementation support

If a future Core Metadata field were defined as a hard compatibility constraint, a resolver could theoretically perform:

```text
candidate
   ↓
Core Metadata available?
   │
   ├── no
   │     ↓
   │   fallback behavior
   │
   └── yes
         ↓
   Requires-Python
         ↓
   implementation support
         ↓
   other candidate constraints
         ↓
   select/download/build
```

This is technically plausible because PEP 658 provides the metadata transport.

However, it introduces a crucial fallback question:

```text
What should happen when implementation metadata
is unavailable?
```

Possible policies include:

```text
accept candidate and try installation
accept candidate but build may fail
download distribution and inspect metadata
treat support as unknown
```

The proposal would need to choose and standardize one.

---

## 9. Absence of metadata must not silently mean universal support

Suppose a hypothetical future release contains:

```text
Supported-Implementation: CPython
```

but an older repository/client path does not expose that metadata before download.

The consumer sees:

```text
implementation support = unknown
```

not necessarily:

```text
implementation support = all
```

This distinction is important.

A safe semantic model would likely distinguish:

```text
declared supported implementations
```

from:

```text
support information unavailable
```

Otherwise repository transport limitations could accidentally become claims about package compatibility.

This is a design issue for the future proposal, not something established by PEP 658.

---

## 10. Sdist relevance

PEP 658 explicitly supports metadata for both wheels and source distributions.

This is important because one of the strongest proposed consumer benefits is avoiding an unnecessary source build.

Conceptually:

```text
PyPy environment
      ↓
sdist candidate
      ↓
Core Metadata available
      ↓
implementation support says:
    CPython only
      ↓
candidate rejected
      ↓
no source build
```

This is precisely the type of workflow that could make release-level implementation metadata useful.

However, the research must establish that:

```text
implementation support is actually the
reason the build should be avoided
```

rather than:

```text
compiler missing
external dependency missing
ABI mismatch
build backend failure
platform problem
```

The metadata transport mechanism cannot distinguish those root causes by itself.

---

## 11. Relationship to `Requires-Python`

`Requires-Python` is already exposed through the Simple API.

For example:

```text
requires-python: >=3.10
```

can be evaluated before downloading the artifact.

This means the ecosystem already has a complete example of:

```text
release/file compatibility information
        ↓
Simple API exposure
        ↓
candidate filtering
```

That is useful precedent.

But it does not prove that an implementation field should receive identical treatment.

The implementation-support field would first need to establish:

1. that it is a normative compatibility constraint;
2. that it applies at the same artifact/release level;
3. that consumers need it during resolution;
4. that its semantics are sufficiently reliable;
5. and that early filtering materially improves installation behavior.

---

## 12. Relationship to PEP 794

PEP 794 is useful structural prior art because it introduces additional release-level Core Metadata fields.

This demonstrates that new release-level information can be standardized in Core Metadata and transported through existing metadata mechanisms.

However:

```text
PEP 794:
    defines release metadata semantics

PEP 658 / PEP 714:
    define transport/access to Core Metadata
```

The transport mechanism does not provide the justification for the semantic field.

A future implementation-support proposal would therefore need:

```text
semantic justification
        +
consumer benefit
        +
transport compatibility
```

rather than relying on PEP 658 alone.

---

## 13. Relationship to PEP 643

PEP 643 is also relevant.

It established the role of static versus dynamic metadata in source distributions and clarified that metadata in an sdist may represent information intended to be retained when the wheel is built.

This matters if implementation support is intended to be a release-level property.

A hypothetical:

```text
Supported-Implementation: CPython
```

would ideally have a well-defined relationship between:

```text
sdist metadata
        ↓
wheel metadata
```

If the value can change depending on the build environment, then it may not actually be a release-level property.

PEP 643 therefore provides a useful test:

> Is implementation support fixed when the release is created, or can it legitimately depend on the environment in which the wheel is built?

That question must be answered before relying on PEP 658 for early filtering.

---

## 14. Metadata availability versus release-level truth

There are two separate questions:

### Transport question

```text
Can the client obtain the metadata before
downloading/building the artifact?
```

PEP 658/714:

```text
yes, when the repository exposes it.
```

### Semantic question

```text
Does the metadata reliably tell the client whether
this release supports the current implementation?
```

PEP 658/714:

```text
not addressed.
```

The proposed PEP must solve the second question.

---

## 15. Security and correctness considerations

If a future implementation-support field becomes a hard candidate filter, incorrect metadata could prevent installation of a package that would actually work.

For example:

```text
Supported-Implementation: CPython
```

might cause a resolver to reject a PyPy installation even if the source would successfully run there.

Conversely:

```text
Supported-Implementation: CPython, PyPy
```

could cause a resolver to select a release that actually fails on PyPy.

Therefore the field would have consequences similar to other normative compatibility metadata.

The proposal would need to consider:

* producer responsibility;
* stale declarations;
* release-to-release changes;
* sdist/wheel consistency;
* missing metadata;
* repository support;
* old installers;
* and incorrect declarations.

PEP 658 does not solve these semantic correctness problems.

---

## 16. Current evidence assessment

| Question                                                                      | Finding           | Confidence |
| ----------------------------------------------------------------------------- | ----------------- | ---------: |
| Can Simple API expose Core Metadata separately?                               | Yes               |       High |
| Does PEP 658 apply to wheels and sdists?                                      | Yes               |       High |
| Is separate metadata availability optional?                                   | Yes               |       High |
| Can clients fall back to downloading the distribution?                        | Yes               |       High |
| Does PEP 714 define current HTML/JSON names?                                  | Yes               |       High |
| Can metadata be inspected before downloading the full artifact?               | Yes, when exposed |       High |
| Does this automatically make every Core Metadata field a resolver constraint? | No                |       High |
| Does it solve implementation-support semantics?                               | No                |       High |
| Could it transport a future implementation-support field?                     | Potentially       |       High |
| Would a new field need fallback semantics?                                    | Yes               |       High |
| Does PEP 658 itself justify a new field?                                      | No                |       High |

---

## 17. Current conclusion

PEP 658 and PEP 714 provide strong **transport prior art** for the proposed idea.

They demonstrate that:

```text
repository
    ↓
distribution candidate
    ↓
Core Metadata
    ↓
consumer
```

can happen without downloading the entire distribution, including for source distributions.

This means a release-level implementation-support field could potentially be useful during candidate selection.

However:

> Metadata transport is not metadata semantics.

PEP 658 and PEP 714 do not establish:

* that implementation support belongs in Core Metadata;
* that implementation support should be a hard resolver constraint;
* what absence of the field means;
* whether the field is release-level or artifact-level;
* or whether the consumer benefit is large enough to justify standardization.

Therefore:

> PEP 658/714 remove one technical obstacle to using a future implementation-support field early, but they do not provide evidence that such a field is necessary.

The remaining research question is:

```text
Does a real release-level implementation-support fact exist
that consumers need before downloading/building a candidate,
and that existing metadata mechanisms cannot express?
```

That remains an empirical question.
