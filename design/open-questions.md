# Open design questions

These are the questions that need to be resolved before a PEP draft
can be written. Community input is welcome on all of them.

---

## 1. Absence semantics

If a release declares `Supported-Implementation: cpython` and `pypy`
is not listed, what should a consumer infer?

**Option A:** PyPy is unsupported — installer should reject.

**Option B:** PyPy status is unknown — installer may warn but should
allow the installation to proceed.

**Option C:** CPython is confirmed; other implementations may work —
no warning needed.

The current preference is **Option B**. Absence means the maintainer
has not made a claim about that implementation, not that it is
incompatible. This avoids stale metadata becoming a hard block as
alternate implementations improve over time.

---

## 2. Field name

The field has been referred to as both `Requires-Implementation` and
`Supported-Implementation`. The name should follow from the semantics
decision above.

Candidates:
- `Supported-Implementation` — positive framing, clearest intent
- `Known-Implementations` — neutral
- `Tested-Implementation` — most precise about what "supported" means

---

## 3. Installer behaviour

Should an unsatisfied declaration (running implementation not listed)
cause the installer to reject the candidate or produce a warning?

`Requires-Python` uses MUST-reject semantics. Given that implementation
compatibility is more fluid than Python version compatibility, the
initial proposal uses SHOULD-warn — allowing alternate implementations
to still attempt installation while surfacing the information.

---

## 4. Wheel interaction

What should happen if a release declares `Supported-Implementation: cpython`
but publishes a `py3-none-any` wheel?

The current view is that these operate at different semantic layers:
the field describes the project/release, wheel tags describe a specific
artifact. A `py3-none-any` wheel says the artifact has no
implementation-specific compiled code; the field says the project has
only been tested on CPython. These are not necessarily contradictory.

---

## 5. Build vs runtime scope

Should the field describe only runtime compatibility, or also build-time
requirements?

The current proposal is runtime-only. Build-time requirements (where
a source build itself fails on a given implementation) may be addressed
separately, potentially through an extension to PEP 725.

---

## 6. Supported-Platform relationship

`Supported-Platform` has existed in Core Metadata since 1.1 (PEP 314)
but its semantics have never been specified. Its documented scope is
OS and CPU for binary distributions.

Should this proposal also clarify or deprecate `Supported-Platform`,
or leave it unchanged?