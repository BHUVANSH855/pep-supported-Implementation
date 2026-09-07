# PEP 625 Notes — Source Distribution Filenames

**URL:** https://peps.python.org/pep-0625/
**Relevance:** High

Conforming sdists use a filename containing distribution name and version,
rather than wheel-style Python/ABI/platform tags.

This is intentional.

An sdist is source, not a completed binary artifact.

## Research relevance

The filename therefore does not provide the same implementation-selection
signal as a wheel filename.

But this should not be described as a flaw.

The actual question is:

> Does another release-level metadata mechanism need to describe support before
> the source is built?

PEP 625 establishes the artifact distinction; it does not answer that
question.
