# PEP 780 Notes — ABI Features

**URL:** https://peps.python.org/pep-0780/

**Relevance:** High

## What PEP 780 addresses

PEP 780 proposes exposing Python interpreter ABI characteristics through a standardized `sys_abi_features` environment marker.

The motivating problem is that implementation identity alone is not always sufficient to determine compatibility.

For example, two environments can both be:

```text id="x8m2qv"
CPython
```

while differing in ABI or interpreter configuration.

PEP 780 addresses this additional compatibility dimension through environment features. ([peps.python.org](https://peps.python.org/pep-0780/))

## Why this matters

Implementation identity is not the complete Python compatibility surface.

Conceptually:

```text id="c4q7nx"
CPython
CPython + free-threading
```

may have the same implementation identity while presenting different compatibility characteristics to extension modules and other software.

Other ABI/environment characteristics can similarly affect compatibility.

Therefore:

```text id="m9w3kp"
implementation identity
```

must not be treated as equivalent to:

```text id="h6r2vz"
complete runtime compatibility.
```

## Correct layering

The current research should distinguish at least:

```text id="y4k8qf"
Requires-Python
    ->
Python language version

implementation identity
    ->
Python implementation

PEP 780
    ->
ABI/environment features

wheel tags
    ->
compatibility of a built artifact

possible release-support metadata
    ->
producer-declared release support
```

These layers can interact, but they do not have identical semantics.

## Implementation support is not ABI support

A hypothetical:

```text id="g5n1wc"
Supported-Implementation: cpython
```

should not silently mean:

```text id="s2q7mx"
all CPython ABI/environment configurations are supported.
```

That would be too broad.

Conversely, a package may support CPython generally while excluding a particular ABI configuration.

That is a different statement.

For example:

```text id="j8v3qa"
Implementation:
    CPython

Additional environment condition:
    free-threaded ABI required/prohibited
```

cannot be represented adequately by an implementation name alone.

## Why PEP 780 is a scope boundary

PEP 780 demonstrates why a proposed implementation-support field should remain narrow.

If a new field were allowed to encode arbitrary environment predicates, it could evolve into a general compatibility expression language:

```text id="b6x9mr"
implementation
+
version
+
ABI features
+
platform
+
build configuration
+
other environment properties
```

That would overlap with several existing packaging mechanisms.

Such expansion would make the semantics harder to specify and could duplicate wheel tags, environment markers, and other compatibility mechanisms.

Therefore a support declaration should not become a general-purpose ABI/environment expression language merely because implementation identity alone is insufficient for every package.

## Guppy3 as a boundary case

Guppy3 is a useful empirical boundary case for this distinction.

Its published support information distinguishes ordinary CPython from free-threaded CPython.

That is valuable evidence that:

```text id="q7m2vc"
"CPython supported"
```

can be insufficient to describe the complete compatibility boundary of a real package.

However, this does **not** automatically establish that the missing information belongs in a `Supported-Implementation` field.

The free-threading distinction is precisely the kind of ABI/environment feature that PEP 780 is intended to make machine-readable.

Therefore Guppy3 should be treated as a boundary/control case:

```text id="e5r8ny"
implementation identity alone
    ->
insufficient for complete compatibility

but:

implementation identity + ABI feature metadata
    ->
potentially sufficient
```

This prevents the research from over-counting ABI restrictions as evidence for a new implementation-support primitive.

## Runtime support versus artifact compatibility

A package may also express some ABI restrictions through wheel tags.

For example, a compiled extension may produce different wheels for different ABI configurations.

This means the research must distinguish:

```text id="p3k7wf"
artifact requires ABI X
```

from:

```text id="w6m1qx"
producer declares release support only under ABI X
```

The former may already be solved by wheel compatibility tags.

The latter is a release-level support-policy question.

Again, the existence of a semantic distinction does not establish that a new field is necessary.

## Interaction with environment markers

PEP 780 is also relevant to dependency markers.

If an ABI feature can be expressed as an environment marker, a package may be able to conditionally select dependencies based on that feature.

This creates another existing-mechanism test:

```text id="h9q2vs"
Can the observed restriction be represented as
an environment-dependent dependency instead?
```

If yes, the case may belong to PEP 508 semantics rather than release-support metadata.

The research should therefore avoid treating every ABI-specific dependency or runtime behavior as evidence for a new field.

## Implementation identity should remain narrow

The strongest conceptual model is:

```text id="s8v4kc"
Supported-Implementation
    ->
implementation identity only
```

while:

```text id="r5q2mw"
ABI/environment constraints
    ->
existing ABI/environment mechanisms
```

and:

```text id="n3j7xp"
artifact-specific compatibility
    ->
wheel tags
```

This keeps each mechanism responsible for a distinct compatibility dimension.

If a release's support boundary genuinely requires a combination of implementation identity and ABI features, that should be investigated as a potential interaction between mechanisms rather than automatically expanding the implementation field itself.

## Consumer-decision test

For an empirical case involving implementation and ABI differences, ask:

```text id="x6w9qp"
1. Is the restriction implementation identity?

2. Is it actually an ABI/environment feature?

3. Can wheel tags already express the artifact restriction?

4. Can environment markers express a dependency condition?

5. Does the release itself make a broader support-policy claim?

6. What consumer decision remains impossible?

7. Would a separate release-level implementation field
   solve that decision without becoming an ABI expression language?
```

Only the final residual category should remain relevant to the proposed field.

## Current conclusion

PEP 780 provides important **complementary prior art and scope control**.

It demonstrates that:

```text id="a2r7mv"
implementation identity
```

is not the complete compatibility surface.

ABI and interpreter-environment features can independently affect package compatibility, including for different configurations of the same Python implementation. ([peps.python.org](https://peps.python.org/pep-0780/))

This strengthens, rather than weakens, the case for keeping a hypothetical implementation-support field narrow.

The proposed concept should not become:

```text id="k5x8nd"
a general-purpose compatibility expression language.
```

Instead, the research should preserve the layering:

```text id="m7q2wb"
Requires-Python
    -> Python version

Supported implementation
    -> producer-declared implementation identity support

PEP 780
    -> ABI/environment features

Wheel tags
    -> built-artifact compatibility
```

**Current conclusion:** PEP 780 is complementary prior art and a scope boundary. It demonstrates that implementation identity alone cannot describe every compatibility condition, while also providing a strong reason not to turn a hypothetical implementation-support field into a general-purpose ABI/environment constraint language.
