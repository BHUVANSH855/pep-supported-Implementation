# Paul Moore — Reply 2

**Author:** pf_moore (Paul Moore, CPython core developer, packaging PEP delegate)
**Date:** September 6, 2026
**Thread:** Requires-Implementation pre-PEP

## Full text

> Currently, no I don't think there is [a standard way for an installer
> to know a package is CPython-only from sdist metadata before building].
> But as I said, I think it's something that should be part of the PEP 725
> discussions, and is only worth raising as its own individual proposal if
> the conclusion is that PEP 725 considers it out of scope (I feel like
> I'd be disappointed if that happened, though).

## Analysis

This is the most important single reply in the thread.

Paul explicitly confirms three things:

1. The gap is real — no standard mechanism currently exists
2. He personally wants PEP 725 to handle it
3. If PEP 725 cannot handle it, a standalone PEP is warranted

The phrase "I'd be disappointed if that happened" reveals his preference
but also signals he accepts the possibility of a standalone proposal.

## Strategic implication

If the PEP 725 authors confirm this is outside their scope, Paul Moore's
own words become the justification for a standalone Requires-Implementation
proposal. Save this reply. It is the bridge.

## Key quote to save

> "it's only worth raising as its own individual proposal if the
> conclusion is that PEP 725 considers it out of scope"

This is Paul giving explicit permission for a standalone PEP under
the right conditions.
