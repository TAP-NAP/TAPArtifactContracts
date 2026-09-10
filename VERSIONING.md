# Versioning and Compatibility

## Independent families

Every public-format and security-schema family owns its own version. The fact
that several current families end in `v1` does not make them interchangeable.

A consumer MUST route by the complete family identifier. It MUST reject an
unknown identifier, a structurally different family, or a manifest/binding pair
whose families do not match. It MUST NOT guess a family from similar fields.

Current families include, but are not limited to:

- Still Photo manifest v1
- Live Photo manifest v1
- TAP Video manifest v1
- Still Photo content-binding v1
- Live Photo content-binding v1
- TAP Video content-binding v1
- App Attest capture-signing v1
- TAP Video depth-registration v1
- TAP Video KLV frame v1
- TAP Video inline calibration v1 (`CALD`, optional KLV extension)
- TAP Video capture-telemetry v1 (optional UUID extension)
- `.tapnap` verification-export v1

## Reviewed contract revisions

A reviewed Git commit identifies one publication of these documents. It is not
a global wire-format version and may document several independent v1 families
at once. A future repository tag MAY name a reviewed commit, but consumers MUST
pin the exact reviewed commit rather than infer a wire-format version from a
tag name.

## Change classification

The following require an explicit family revision unless the existing contract
already permits them:

- adding, removing, renaming, or changing a required field;
- changing `null` versus omitted behavior;
- changing an enumeration, unit, coordinate system, matrix order, or time base;
- changing canonical JSON bytes, hash input, resource ordering, or encoding;
- changing proof-slot size, binary layout, container location, or excluded
  byte range;
- changing which resources are bound or which schema-family combinations are
  accepted.

Editorial clarification is non-breaking only when it does not change any
producer byte or consumer decision. If an apparent clarification reveals that
an implementation differs, that downstream repository must record the mismatch
without redefining this contract. Changing the shared decision still requires
an owner-approved compatible or breaking family revision.

## Current pre-release policy

Before the first public release, `.tapnap` verification-export v1 is explicitly
converged on the Still/Live Photo and TAP Video layouts in the current
[transport definition](transport/tapnap-v1.md). Its sidecar identifier and
`version: 1` remain unchanged despite adding the `tapVideo` package kind,
`primaryVideo` role, and video-specific trust-boundary text. This is a bounded
pre-release exception to the enumeration-change rule above, not an editorial
claim that earlier photo-only revisions could handle video packages. Producers
and consumers must adopt the same reviewed contract revision. Unknown kinds,
roles, sidecar fields, and mismatched resource families still fail closed.
The signed media's manifest, binding, proof, and container families do not change.

The optional [`CALD`](containers/tap-video-container-v1.md#inline-calibration-extension-cald)
and [`TAPCAMTELEMETRY1`](containers/tap-video-capture-telemetry-v1.md)
extensions are adopted at their existing v1 definitions. This pre-release
adoption does not change their bytes or semantics, `TVER=1`, manifest and
content-binding families, or any schema-version value. Consumers adopt them by
pinning the reviewed contract commit; no family version is advanced.

Superseded development identifiers and cross-family combinations are
unsupported. Consumers fail closed instead of silently accepting a legacy alias
or coercing one v1 family into another.
