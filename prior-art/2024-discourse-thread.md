# Prior discussion: "Python implementation in metadata" (January 2024)

**URL:** https://discuss.python.org/t/python-implementation-in-metadata/42653
**Date:** January 7-10, 2024
**Status:** Closed, no action taken

## Summary

A user asked why Python implementation is not stored in package metadata,
with the goal of being able to force PyPy usage.

## Key responses

**Paul Moore:** Questioned why a library would need to block other
implementations. Suggested classifiers and wheel tags as the
appropriate existing mechanism.

**Sinoroc:** Suggested Trove classifiers.

**C.A.M. Gerlach:** Pointed to classifiers, wheel tags, and
backend/plugin approaches as existing partial mechanisms.

## How this differs from the current proposal

The 2024 thread was about a user wanting to FORCE a particular
implementation in a development environment.

The current proposal is about a PACKAGE declaring its own runtime
compatibility as a normative constraint for installer candidate
selection — a different semantic layer, especially for sdists.

## How to cite this in discussions

Do NOT say "the community said classifiers were inadequate."
Paul Moore and C.A.M. Gerlach argued classifiers were appropriate
for the use case discussed (forcing an implementation).

DO say: "The 2024 discussion addressed a user's desire to force
an implementation. The current proposal addresses package-level
compatibility declaration for installer use — a different problem."

## Why this thread does not block the current proposal

- No PEP was proposed or rejected
- No PR was opened
- The use case discussed was different
- The thread closed with no normative resolution
