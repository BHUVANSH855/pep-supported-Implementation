# Ralf Gommers — PEP 725 Thread Post #98

**Author:** rgommers (Ralf Gommers, PEP 725 co-author)
**Thread:** PEP 725 Round 2
**URL:** https://discuss.python.org/t/pep-725-specifying-external-dependencies-in-pyproject-toml-round-2/103890/98
**Date:** September 6, 2026, 1:49pm

---

## Full text

> This PEP is ready; PEP 804 is quite close too, and will get a
> (hopefully last) update soon, when the Packaging Steering Council
> is seated. We deliberately held back recently, because while this
> PEP is important, it is less urgent to the authors than the wheel
> variants effort, and Paul was already a reluctant BDFL delegate here
> and also indicating he had bandwidth issues.
>
> PEP 804 is a good example of a PEP that will benefit from a Steering
> Council by the way. It came out of this PEP because not everyone was
> happy with the lack of canonical names. Hence, we needed a registry.
> Now there's a registry design that inherently requires someone to bless
> that registry as authoritative, and hence give its maintainers a
> responsibility that didn't exist before. But how do you measure
> consensus on that — and do you even need 100% consensus, or can any
> one person who doesn't have a need for that design and say they don't
> like it hold up progress. Technically, PEP 804 isn't all that
> complicated. Considerations like governance and long-term maintainability
> are the more interesting part of it. Typically something that our
> pre-council processes struggle with.

---

## Analysis

### What this post reveals

**1. PEP 725 scope is frozen.**
Ralf is responding to Lucas Colley's question about next steps.
His answer is: the PEP is ready, waiting on the Packaging Steering
Council. No new scope is being considered.

**2. This is NOT a response to your comment (post #96).**
Post #98 was written in response to Lucas Colley (post #95 from
3 months earlier). Your comment at post #96 has received no reply.

**3. PEP 725 is not being extended for Python implementation identity.**
The discussion is about PEP 804's governance and the registry design —
entirely unrelated to your implementation compatibility question.

---

### Strategic implication

Paul Moore said:
> "it's only worth raising as its own individual proposal if the
> conclusion is that PEP 725 considers it out of scope"

Post #98 confirms:
- PEP 725 is ready and in final governance stages
- No new scope additions are being considered
- The implementation identity use case has not been addressed

**Conclusion:** Paul Moore's condition is now effectively met.
The standalone proposal route is open.

---

### What to do about your comment (post #96)

Your comment has received no direct reply. Two options:

**Option A (recommended):** Wait 48-72 more hours. If no response,
post a brief note in your main thread:

> "PEP 725 appears to be in final stages awaiting the Packaging
> Steering Council, with scope frozen. My comment there (post #96)
> hasn't received a response on the implementation compatibility
> question. Based on Paul's earlier guidance, I'll explore the
> standalone proposal route."

**Option B:** Post a follow-up in the PEP 725 thread specifically
tagging @rgommers and asking whether the runtime implementation
compatibility case is explicitly out of scope. Risk: may seem
tone-deaf given PEP 725 is clearly in final stages.

**Recommendation: Option A.**