# PEP 421 Notes — `sys.implementation`

**URL:** https://peps.python.org/pep-0421/

**Relevance:** High

PEP 421 standardized `sys.implementation` as a source of interpreter implementation identity and related implementation details.

## What PEP 421 establishes

The important attribute for this research is:

```text
sys.implementation.name
```

This provides the runtime's implementation identifier.

PEP 421 also defines:

```text
sys.implementation.version
```

which represents the version of the Python implementation itself, rather than necessarily the version of the Python language that it implements.

This distinction matters for alternative implementations: implementation identity and Python language version are separate dimensions.

The current Python documentation continues to expose `sys.implementation` as interpreter-specific information maintained by the runtime.

## Research relevance

The environment side of the proposed problem is therefore already standardized.

Conceptually:

```text
Environment:

    sys.implementation.name
    sys.implementation.version
```

The unresolved research question is on the distribution side:

```text
Release:

    which implementation identities does the producer support?
```

These are deliberately different questions.

For example:

```text
sys.implementation.name == "cpython"
```

is a statement about the interpreter executing code.

It is not, by itself, a statement that an installed distribution release claims support for CPython.

## Important boundary

PEP 421 should therefore be treated as **implementation-identity prior art**, not as evidence that a new support field is required.

It does not define:

* distribution metadata;
* package support policy;
* resolver eligibility;
* whether a distribution is installable on an implementation;
* whether a project has tested an implementation;
* whether a project guarantees support for an implementation.

Consequently, a proposal should not present `sys.implementation` as the missing mechanism.

The existing mechanism already answers:

```text
What implementation am I running?
```

The proposed metadata, if ultimately justified, would answer a different question:

```text
What implementation does this particular distribution release claim to support?
```

## Relationship to packaging markers

Packaging already exposes implementation identity through environment markers such as:

```text
implementation_name
implementation_version
platform_python_implementation
```

Those markers allow dependency specifications to be conditional on the target environment.

This is useful prior art, but it does not automatically solve release-level self-support.

For example:

```text
implementation_name == "pypy"
```

can determine whether a conditional dependency applies.

It does not currently mean:

```text
this distribution release supports PyPy
```

Therefore the existence of implementation markers should be considered when evaluating the need for new metadata, but should not be treated as equivalent semantics.

## Relationship to `Requires-Python`

PEP 421 also reinforces the distinction between:

```text
Python language version
```

and:

```text
Python implementation identity
```

`Requires-Python` addresses the former.

Conceptually:

```text
Requires-Python
    -> Which Python language versions are compatible?
```

while a hypothetical implementation-support declaration would address:

```text
Supported implementation
    -> Which Python implementations does this release claim to support?
```

This is a genuine semantic distinction.

However, the existence of that distinction is not sufficient evidence that a new Core Metadata field is necessary. The research must still demonstrate a concrete consumer decision that existing metadata cannot make.

## Exact identity versus compatibility

A future support declaration must not introduce an ambiguous meaning for implementation names.

For example:

```text
Supported-Implementation: CPython
```

could potentially be interpreted as either:

```text
sys.implementation.name == "cpython"
```

or:

```text
any runtime sufficiently compatible with CPython
```

These are not equivalent.

The first is a deterministic identity comparison.

The second introduces an additional compatibility relation that would itself need to be defined and standardized.

For machine-actionable metadata, exact implementation identity is therefore the stronger initial model.

## CPython-derived runtimes

The distinction becomes particularly important for CPython-derived or CPython-compatible runtimes.

A producer may mean any of the following:

```text
A. Tested on CPython.

B. Supports runtimes whose implementation identity is
   exactly "cpython".

C. Supports runtimes that are behaviorally compatible
   with CPython.
```

Those statements have different semantics.

PEP 421 provides the identity primitive needed for A/B-style machine-readable reasoning, but it does not define C.

Therefore a future metadata specification should not silently equate implementation identity with behavioral compatibility.

## Implementation identity is not artifact compatibility

Wheel tags already provide implementation-related information for individual artifacts.

This produces another important boundary:

```text
sys.implementation
    -> identity of the running interpreter

wheel tags
    -> compatibility of a particular artifact

hypothetical support metadata
    -> producer's release-level support declaration
```

These mechanisms may use related implementation concepts without being interchangeable.

In particular, the fact that a wheel is tagged generically does not automatically mean that the producer supports every Python implementation.

Conversely, the absence of an implementation-specific wheel does not automatically establish that the release is unsupported on that implementation.

## Implication for the proposed metadata

If research eventually demonstrates that a release-level support declaration is necessary, the safest conceptual relationship would be:

```text
Supported implementation identity
        ↓
defined using the existing implementation vocabulary
        ↓
sys.implementation.name
```

rather than introducing a second vocabulary such as:

```text
CPython
PyPy
GraalPy
...
```

with independently defined semantics.

This would reduce the risk of having two competing definitions of Python implementation identity.

The field would still need to specify:

* exact matching semantics;
* whether implementation versions can be constrained;
* interaction with `Requires-Python`;
* interaction with wheel tags;
* interaction with environment markers;
* meaning of omission;
* treatment of compatibility runtimes;
* behavior when producer declarations are stale or incorrect.

## Evidence versus conclusion

The following should **not** be inferred from PEP 421 alone:

```text
sys.implementation exists
        ↓
therefore Core Metadata needs Supported-Implementation
```

That conclusion does not follow.

The defensible inference is:

```text
sys.implementation exists
        ↓
a standardized implementation identity already exists
        ↓
a new support declaration need not invent an identity vocabulary
        ↓
the remaining question is whether release-level support
is a missing packaging semantic.
```

That final question requires the real-world residual-case and consumer-decision evidence being collected elsewhere in this research.

## Current conclusion

PEP 421 provides strong prior art for the **environment identity** side of this problem.

The ecosystem already has a standardized way to identify the running implementation:

```text
sys.implementation.name
sys.implementation.version
```

Therefore this research should **not propose a new implementation identity mechanism**.

The important unresolved distinction is:

```text
Environment identity
    !=
Release support declaration
```

If a future Core Metadata field is justified, it should define a precise relationship to the existing implementation identity rather than inventing a parallel vocabulary.

The existence of `sys.implementation` is therefore a **design constraint and enabling primitive**, not evidence by itself that a new Core Metadata field is necessary.
