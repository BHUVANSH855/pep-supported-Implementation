# Next Reply Draft

**Status:** HOLD — do not post until Paul's Reply 8 is read.

---

## Context for this draft

This reply is to Daniel Diniz (post 9) and acknowledges Paul (post 8,
content not yet captured). It advances the semantic shift and asks
the absence-semantics question publicly.

---

## Draft reply

---

Daniel, the positive-assertion framing is a better way to think about
this — "these implementations are known to work" is more honest and
more stable than "all others are blocked." That directly addresses
the staleness concern.

It does raise the absence-semantics question though. If a distribution
declares `cpython` as supported and `pypy` is absent, should a consumer
treat that as "PyPy is unsupported" or just "PyPy status is unknown"?

For the installer use case, the first is most useful. For the ecosystem
tooling case you describe, either works. But for the staleness concern,
only the second is safe.

My instinct is "unknown" — the field declares what the maintainer has
confirmed, not what is technically impossible. That would make it
informational with a SHOULD-warn rather than a MUST-reject, which is
safer but less powerful for installer candidate selection.

On the PEP 725 side: post #98 from Ralf suggests PEP 725 is in final
stages awaiting the Packaging Steering Council, with scope frozen. My
comment at post #96 has not received a direct reply on the implementation
question. Based on Paul's earlier guidance, I'll explore the standalone
proposal route if that holds.

---

## Notes on this draft

- Acknowledges Daniel's semantic reframing without over-crediting him
- Asks the absence-semantics question directly — key design issue
- Does not commit to MUST-reject vs SHOULD-warn
- Does not re-open RestrictedPython or wheel debate
- Notes PEP 725 status briefly, consistent with earlier commitment
- ~150 words — right length for current thread state

## What to adjust after reading Paul's Reply 8

- If Paul raised new objections: address them first, then Daniel
- If Paul was positive: reference his point briefly before Daniel's
- If Paul changed the strategic landscape: revise this draft significantly

**Do not post until Paul's Reply 8 is confirmed.**