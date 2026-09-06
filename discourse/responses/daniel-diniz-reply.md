# Daniel Diniz — Reply (Post 9)

**Author:** devdanzin (Daniel Diniz)
**Date:** September 6, 2026
**Thread:** Requires-Implementation pre-PEP

---

## Full text

> Yes, but I see value in knowing beforehand whether there are known
> issues that would make it (temporarily?) impossible to work. That is
> a common situation for alternative implementations: they'd like to
> support a given extension, but due to priorities, bandwidth, size of
> refactoring, etc. cannot for many years.
>
> But version Y of PyPy will never retroactively support a given extension,
> so maybe the way to avoid staleness would be to add versions to the
> classifier? E.g. "doesn't support PyPy<7.20.x"?
>
> I think the better objective is to list what is known to be supported,
> and that should usually be evergreen. IMHO the positive assertion helps
> when picking the extension stack to build your product from, just like
> "pure Python" or "cffi" used to help deciding whether a given project
> could be done in PyPy. Also gives a strong signal on implementation
> diversity on the Python ecosystem.
>
> I do have one personal use case that's niche enough to not matter.
> I fuzz interpreters from different Python implementations and extensions
> built for these interpreters. Programmatically figuring out whether a
> given extension supports a given implementation beats attempting to build
> to find out. I could then fuzz extension X on CPython, PyPy and RustPython
> without trial-and-error to assess support.
>
> I also have a project that downloads thousands of extensions, builds them
> and runs their test suites under different implementations (currently
> focused on FT support, but PyPy is a future target). The system figures
> out whether a given extension is supported, but having that information
> encoded in metadata would help.
>
> But as I said, the N of people with this need is 1 or very near to it.
> But If N is (slightly) >1, @BHUVANSH855 is very likely to be the other
> person who would be helped by this for that end, as he develops software
> that analyzes CPython and PyPy differences in extension/package code.
>
> Full disclosure: I work with Bhuvansh on some projects where this would
> be useful, though I didn't know about this idea until reading this thread.

---

## Analysis

### What Daniel contributed

**1. The semantic reframing — most important contribution.**
A positive "known to be supported" declaration is more defensible
than a negative exclusionary constraint. This shifts the design from
`Requires-Implementation` to `Supported-Implementation` and directly
addresses Ralf's stale-metadata objection.

**2. Two concrete non-installer use cases:**

- **Interpreter fuzzer:** knowing which implementations are supported
  before attempting a build eliminates trial-and-error across CPython,
  PyPy, and RustPython.

- **Large-scale compatibility tester:** a system downloading thousands
  of extensions and running test suites under different implementations
  would pre-filter by declared support rather than discover it by building.

**3. Ecosystem diversity signal:**
Machine-readable support declarations across PyPI would allow tooling
to build a picture of implementation diversity without trial-and-error.

---

### The risks in Daniel's reply

**Risk 1: "The N of people with this need is 1 or very near to it."**
This is the most damaging sentence in the thread. It provides ammunition
for dismissing the proposal as too niche to warrant standardisation.

**Risk 2: Working relationship disclosure.**
Daniel disclosed he works with the proposer. Combined with N=1, the
community may read this as: one proposer, one collaborator, no
independent validation. Do not lean on Daniel as independent evidence.

---

### What to use from this reply

**Use:**
- The semantic reframing (positive assertion, not exclusionary constraint)
- The absence-semantics question raised implicitly
- The non-installer tooling use cases as a category of need

**Do not use:**
- Daniel as independent validator (relationship disclosed)
- "N of people with this need is 1" as anything positive
- His versioned-classifier idea without further research

---

## Impact on proposal

**Before post 9:** `Requires-Implementation: cpython` — hard exclusionary
**After post 9:** `Supported-Implementation: cpython` — positive declaration

See `design/semantic-shift.md` for the full documented change.