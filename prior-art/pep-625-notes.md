# PEP 625 Notes

## Purpose

PEP 625 standardizes source distribution filenames.

## Relevant format

A conforming sdist uses:

```text
{distribution}-{version}.tar.gz
```

The filename communicates the distribution name and version.

## Important limitation

Unlike a wheel filename, the sdist filename does not encode:

- Python implementation;
- Python version;
- ABI;
- platform.

This is intentional.

An sdist represents source that can potentially be built for a target
environment rather than a completed binary artifact.

## Relationship to the research

This creates an important artifact-level distinction:

```text
wheel
    implementation/version/ABI/platform tags

sdist
    distribution/version filename
```

The absence of an implementation tag in the sdist filename is not itself a
bug.

However, it means that implementation compatibility cannot be inferred from
the standardized sdist filename in the same way it can be inferred from a
wheel tag.

## Current conclusion

PEP 625 supports the distinction between source-distribution identity and
built-artifact compatibility.

A future release-level implementation metadata mechanism would therefore
need to live somewhere other than the current sdist filename format.
