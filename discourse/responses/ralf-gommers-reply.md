# Ralf Gommers — Reply

**Author:** rgommers (Ralf Gommers)
**Role:** NumPy/SciPy core developer, PEP 725 co-author
**Date:** September 6, 2026
**Thread:** Requires-Implementation pre-PEP

## Full text

> Do you actually have a real-world need for this? You say "a gap that
> I keep running into", but there is nothing concrete you mention and
> you can't really fix much here beyond getting a better error message.
> What specifically do you keep running into?
>
> I'll also note that this is your second proposal in a week for new
> Core Metadata (a security field being the other one, see here). It
> all feels a little premature (and AI-driven, as pointed out in the
> other thread).
>
> It seems to me that this is a lot like upper bounds on python-requires,
> except the need being less clear. The recommended approach today for
> that one is: raise a clear error at the top of your build config file,
> so the user is clear on why the build fails - and not just "PyPy is
> not supported" but why that's the case, or link to a tracking issue.
>
> An installer can't do much better: if a package is in a dependency tree,
> it is needed, so an error message it is. An installer will give an error
> message a bit earlier, but it'll be more generic.
>
> I have to say I don't really understand the link to PEP 725. If the
> problem would have been "the package does build and work under an
> alternative interpreter, but a build dependency is missing" then yes
> sure. But that wasn't the question here.
>
> Which I don't even think is a well-defined situation. Being compatible
> with CPython is a key goal of alternative interpreters like PyPy. They
> succeed in many cases, and when they don't, it's often a bug or missing
> feature on their side that's fixable. If package metadata would prevent
> them from even trying to install, it just makes their jobs harder.
>
> Concrete example here: there's some effort ongoing to make PyPy
> understand abi3 wheels. Say that works, that'd be a nice achievement.
> Packages that declare "doesn't support PyPy" would then be unhelpfully
> out of date.

## Analysis — claim by claim

### Claim 1: No real-world need demonstrated
**Valid.** The original post was vague on personal motivation.
PyRift is the concrete personal motivation — working across CPython
and PyPy reveals the absence of machine-readable compatibility signals.

### Claim 2: AI-driven
**Serious credibility challenge.** Must be addressed directly and calmly.
AI tools were used for research (prior art, PEP landscape) not to
generate the proposal's substance. PyRift is the real motivation.

### Claim 3: Stale metadata (abi3/PyPy example)
**The strongest technical objection in the entire discussion.**
If PyPy gains abi3 wheel support, packages declaring cpython-only
become unnecessarily restrictive. This is NOT yet answered.

Candidate responses to develop:
- The field is per-release, not permanent. foo 1.0 can be cpython-only
  while foo 1.1 adds pypy. Metadata does not go stale if treated as
  a release-level declaration.
- Requires-Python has the same staleness property and is accepted.
- The field should describe "tested and supported" not
  "technically impossible forever."

### Claim 4: Does not understand PEP 725 link
**Ralf (PEP 725 co-author) agrees this is outside PEP 725 scope.**
This actually supports the standalone proposal route. Save this.

## Priority actions from this reply

1. Answer the stale metadata objection — this is blocking
2. Provide concrete real-world cases beyond RestrictedPython
3. Address the AI concern directly in thread reply
4. Note that Ralf implicitly confirms PEP 725 does not cover this case

## The stale metadata objection — status

UNRESOLVED. This is the primary open question before the proposal
can proceed. See design/open-questions.md Q2.
