# Eli Schwartz — Reply

**Author:** eschwartz (Eli Schwartz)
**Date:** September 6, 2026
**Thread:** Requires-Implementation pre-PEP

## Full text

> But PEP 425 does, and your original post even pointed this out and
> said that you only care about [sdists]. Why do you now say that you
> actually care about wheels, and cite the example of a project that
> is not utilizing implementation-compatible wheel selection? Report a
> bug to RestrictedPython for distributing incorrectly-built wheels.
>
> ...and a bug to the packaging project, since packaging.tags is also
> broken — per packaging.tags does not support *-none-any wheels for
> non-py* interpreter tags (Issue #311) it does not provide
> "cp3-none-any", or indeed any implementation-specific tag other than
> pp3 which was added after the fact as a special case (odd).

## Analysis

Eli makes two distinct points:

**Point 1:** RestrictedPython is a bad example because its wheel is
incorrectly tagged. The fix is to file a bug with RestrictedPython,
not create new metadata. This point is CORRECT. The example was conceded.

**Point 2:** packaging.tags #311 shows the tooling for implementation-
specific none-any wheels is historically broken. cp3-none-any is not
generated. This point actually STRENGTHENS the proposal: if "just use
wheel tags" requires tooling that is incomplete, the answer is not yet
complete.

## What was correctly conceded

The RestrictedPython wheel example was dropped. This was the right move.
Do not re-use RestrictedPython as a wheel-tagging example.

## What was NOT conceded and should not be

The sdist case remains valid. Eli's objection was specifically about
the wheel example, not about the sdist gap.

## Key evidence Eli provided

packaging.tags Issue #311:
https://github.com/pypa/packaging/issues/311

This is useful counter-evidence when reviewers say "just use wheel tags"
for pure-Python implementation-specific packages.
