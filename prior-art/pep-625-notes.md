# PEP 625 Notes — Source Distribution Filenames

**URL:** https://peps.python.org/pep-0625/

**Status:** Final

**Relevance:** High

## What PEP 625 does

PEP 625 standardizes the filename of a Python source distribution.

A conforming sdist filename has the form:

```text id="3h7v5x"
{distribution}-{version}.tar.gz
```

where the distribution name and version are normalized according to the applicable packaging specifications. The filename therefore communicates the distribution identity and release version without requiring the archive to be downloaded and inspected.

The current Source Distribution Format specification continues to require this filename form and requires the name and version in the filename to match the metadata contained in the archive.

## Why sdists differ from wheels

A source distribution is source code intended to be built into an installable artifact.

Unlike a wheel, its filename does not contain:

```text id="6v7s2c"
python tag
abi tag
platform tag
```

Instead, its standardized filename contains only:

```text id="7u4w8m"
distribution
version
```

This is intentional.

PEP 625 standardizes the sdist filename as a way to identify the distribution and version. It does not attempt to turn the filename into a source-build compatibility description.

Therefore:

```text id="g1c5zr"
example-1.2.tar.gz
```

does not communicate from its filename whether the source can successfully build or run on:

```text id="2s8q4j"
CPython
PyPy
GraalPy
...
```

## Why the distinction matters

This creates an important asymmetry:

```text id="x8q0df"
Wheel:
    filename can communicate artifact compatibility

Sdist:
    filename communicates distribution identity and version
```

That does **not** mean the sdist filename is deficient.

PEP 625 deliberately solves a narrower problem.

The relevant research question is instead:

```text id="k3j9wm"
Before an sdist is built, is there useful
implementation-support information that a
consumer needs but cannot obtain from
existing metadata?
```

That is the residual problem that this research must test.

## PEP 625's pre-build information principle

PEP 625 is particularly relevant because its motivation explicitly concerns avoiding unnecessary processing of an sdist.

Before PEP 625, tools could not always safely determine the distribution name and version from an sdist filename. As a result, they sometimes needed to download the archive and generate or inspect metadata to verify those values.

PEP 625 standardized the filename so tools can obtain those particular facts without downloading, unpacking, or processing the archive.

This establishes useful prior art for a broader packaging principle:

```text id="q2j4ax"
If a consumer can obtain a required fact
from standardized distribution metadata early,
it may be possible to avoid an unnecessary
download, build, or execution step.
```

However, PEP 625 does not establish that implementation support is such a fact.

That remains an empirical question.

## What PEP 625 does not solve

PEP 625 does not provide:

```text id="1m9x3c"
implementation identity
ABI compatibility
platform compatibility
build compatibility
runtime support policy
```

for an sdist filename.

It also does not claim that every source distribution is buildable on every Python implementation.

Those questions are outside the filename specification.

Consequently, this would be an incorrect inference:

```text id="p5f3nk"
sdist filename lacks implementation tags
        ↓
therefore sdist metadata needs implementation tags
```

The correct inference is narrower:

```text id="x2s8dw"
sdist filename lacks implementation compatibility information
        ↓
determine whether consumers actually need that information
        ↓
determine whether existing metadata/build mechanisms provide it
        ↓
only then consider new metadata
```

## Relationship to wheel tags

PEP 425 provides implementation-aware compatibility tags for built distributions.

PEP 625 deliberately does not add equivalent tags to sdist filenames.

This creates a useful boundary:

```text id="b6x1pw"
Wheel filename
    -> built-artifact compatibility

Sdist filename
    -> distribution identity + release version
```

The difference follows from the different nature of the artifacts.

A wheel has already been built for a particular compatibility environment.

An sdist has not.

Therefore implementation-specific information may need to be evaluated at build time for an sdist even when the eventual wheel would have implementation-specific tags.

## The source-build problem

The most relevant potential use case is therefore:

```text id="r8w4kq"
Target environment
        ↓
candidate sdist
        ↓
build attempted
        ↓
implementation-specific failure
```

A hypothetical release-support declaration could potentially move some decisions earlier:

```text id="m7q1sc"
Target implementation
        ↓
release metadata
        ↓
known unsupported
        ↓
avoid source download/build
```

That would be a meaningful consumer benefit if all of the following were true:

1. the implementation restriction is known before building;
2. the producer's declaration is trustworthy enough to use;
3. existing metadata cannot express the restriction;
4. the installer can obtain the metadata before starting the build;
5. rejecting or skipping the release is the desired behavior;
6. the declaration applies to the release rather than merely one build configuration.

These conditions must be demonstrated rather than assumed.

## PEP 625 does not establish that support metadata is required

PEP 625 establishes a useful precedent for exposing release information before expensive processing.

It does **not** establish that implementation support is the missing information.

Other mechanisms may already solve particular source-build restrictions, including:

```text id="j4p9cv"
- Requires-Python;
- dependency environment markers;
- build-system requirements;
- host requirements;
- external dependency declarations;
- build configuration;
- wheel compatibility tags after building;
- project-specific build logic.
```

The research must therefore determine whether the residual source-build cases survive those existing mechanisms.

## Relationship to PEP 643 and static metadata

PEP 625's motivation also interacts with the broader evolution of sdist metadata.

PEP 625 notes that standardized sdist metadata can provide trustworthy distribution information, but argues that filename-level information remains useful in situations where a consumer only has access to the filename or where obtaining the archive would be unnecessarily expensive.

The current source-distribution specification requires an sdist to contain `PKG-INFO` with Core Metadata and a `pyproject.toml`, making standardized metadata available inside the archive.

This reinforces an important distinction:

```text id="v5q0sn"
metadata availability inside an archive
        !=
metadata availability before downloading that archive
```

If a future implementation-support field were added to Core Metadata, that would improve pre-build knowledge **only where the metadata itself is already accessible**.

For an sdist selected from an index, the field would generally not be available merely from the standardized sdist filename.

That limitation must be included in any claim that new metadata would prevent unnecessary source builds.

## Index-level implications

This also raises a critical consumer-path question.

Suppose an index contains:

```text id="w4h7cz"
example-1.2.tar.gz
```

and the release's Core Metadata says:

```text id="u0m5kp"
Supported-Implementation: cpython
```

If the index only exposes the filename and does not expose the release metadata separately, an installer cannot use that field without obtaining the sdist or another metadata representation.

Therefore a proposed support field does not automatically solve:

```text id="r1z8ny"
"Can I know this before downloading the sdist?"
```

The research must distinguish:

```text id="e5x0qk"
metadata exists
```

from:

```text id="v8s2mc"
metadata is available early enough for the
consumer decision we care about
```

This is especially important because the proposed benefit may depend on avoiding a source download or build.

## Sdist versus wheel candidate selection

A realistic installer decision may look more like:

```text id="f3c7qd"
Release 1.2
    |
    +-- compatible wheel
    |
    +-- sdist
```

For a wheel candidate, compatibility tags already provide a strong artifact-level filtering mechanism.

For an sdist candidate, the installer may need to determine whether building the source is viable.

The research question therefore becomes:

```text id="p7y4sa"
Can release-level implementation support metadata
provide a useful decision before the sdist build,
where existing metadata cannot?
```

That is a much narrower and more testable proposition than claiming that sdists need wheel-like implementation tags.

## Relationship to failed-build caching

Another important alternative is avoiding repeated builds through build-failure caching or equivalent local knowledge.

For example:

```text id="s2m9wd"
first attempt:
    sdist build fails on PyPy

later attempt:
    cached failure avoids repeating the build
```

This solves a different problem from producer-declared support:

```text id="c8k1vf"
support metadata
    -> declarative prediction

failed-build caching
    -> empirical knowledge
```

Neither mechanism should automatically be treated as a replacement for the other.

However, failed-build caching is a serious alternative when the motivating problem is primarily repeated build cost rather than the absence of producer support information.

## Residual-case test for sdists

A source-distribution case should therefore remain a serious candidate only when the evidence shows:

```text id="y9n4cs"
1. Exact release identified.

2. Sdist is the relevant candidate.

3. The release has a genuine implementation-specific
   support or compatibility boundary.

4. The boundary is known before the source build.

5. Requires-Python is insufficient.

6. PEP 508 dependency markers are insufficient.

7. Build/host requirements are insufficient.

8. The restriction is not merely an artifact-level
   wheel compatibility issue.

9. The relevant metadata is available early enough
   for the desired consumer decision.

10. The consumer would materially benefit from
    avoiding the source build.

11. Existing alternatives such as failure caching
    do not adequately solve the identified problem.
```

This is substantially stronger evidence than simply observing:

```text id="g2m5hx"
"sdists don't have implementation tags."
```

## Current conclusion

PEP 625 establishes the distinction between source-distribution filenames and wheel compatibility filenames.

It does **not** indicate that sdist filenames should acquire Python implementation tags.

Its more important contribution to this research is methodological:

> standardized metadata can sometimes allow packaging tools to make decisions without downloading or processing an artifact.

For implementation support, however, the research must demonstrate that:

```text id="n3p8jw"
release-level implementation information
```

is both:

```text id="c7r1hx"
needed
```

and:

```text id="z0v5mk"
available early enough
```

to improve a concrete installer, index, or build decision.

**Current conclusion:** PEP 625 establishes the artifact distinction and provides useful prior art for early metadata-based decisions, but it does not itself demonstrate a need for implementation-support metadata. The strongest remaining question is whether an sdist can present a genuine, pre-build implementation-support boundary that existing metadata and build mechanisms cannot communicate early enough for a useful consumer decision.
