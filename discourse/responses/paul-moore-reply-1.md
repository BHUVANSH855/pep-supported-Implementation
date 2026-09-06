# Paul Moore — Reply 1

**Author:** pf_moore (Paul Moore, CPython core developer, packaging PEP delegate)
**Date:** September 6, 2026
**Thread:** Requires-Implementation pre-PEP

## Full text

> If we put aside the case of building a sdist that is known to fail
> on the target interpreter, are there any other use cases for this
> proposal? I can't think of any, to be honest.
>
> And as far as sdists are concerned, this sounds like something that
> would be better handled using PEP 725, in particular the
> external.build-requires and/or external.host-requires fields in
> pyproject.toml, combined with a DepURL which can identify the Python
> implementation available in the build/host environment.
>
> At a minimum, I suggest that you take your use case(s) to the PEP 725
> discussion thread (which admittedly has been idle for a while), and
> explore there whether they can be handled with that proposal. If not,
> you can come back to this proposal with a much better argument for why
> you need specific metadata just to cover the target implementation.

## Analysis

- Paul acknowledges the sdist gap is worth discussing
- He redirects it to PEP 725 as the preferred venue
- He does not reject the proposal outright
- His PEP 725 suggestion is about build-requires — Core Metadata: N/A
  in current PEP 725. This is a gap in his suggestion.
- He explicitly says: "come back with a much better argument" if
  PEP 725 considers it out of scope. This is an invitation, not a rejection.

## What not to concede based on this reply

Do not concede that PEP 725 already solves this. It does not.
Do not concede that build-requires in PEP 725 is equivalent to a
release-level Core Metadata constraint. It is not.
