# Versioning and Compatibility

## Independent families

Every public-format and security-schema family owns its own version. The fact
that several current families end in `v1` does not make them interchangeable.

A consumer routes by the complete family identifier, without guessing from
similar fields. Verification requires supported, matching manifest, binding,
and proof families. An unsupported media extension affects decoding; its covered
bytes still participate in verification.

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

A reviewed Git commit identifies a publication of these documents, not a global
wire-format version. Consumers pin that commit to adopt the documented behavior
across the independent families.

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

An editorial clarification changes neither producer bytes nor consumer decisions.
An implementation mismatch does not redefine the contract; a shared behavior
change requires owner approval and a reviewed revision.

## Current pre-release policy

The following owner-approved changes are adopted before the first public release
without advancing version numbers. Producers and consumers adopt the reviewed
contract commit together:

- [`.tapnap` v1](transport/tapnap-v1.md) includes Still/Live Photo and TAP Video
  packages. Safe paths and unambiguous recognized resource roles are required;
  unsigned descriptive fields and unknown roles do not determine authenticity.
- Encoding/decoding and signature acceptance are separate. Content properties
  and decoder support no longer gate signing or verification. Existing producer
  field representations, identifiers, hash inputs, proof layouts, and canonical
  signing-message algorithms remain unchanged. This changes acceptance behavior,
  not merely editorial wording.
- Optional [`CALD`](containers/tap-video-container-v1.md#inline-calibration-extension-cald)
  and [`TAPCAMTELEMETRY1`](containers/tap-video-capture-telemetry-v1.md) extensions
  are adopted with their existing bytes and semantics, including `TVER=1`.

Superseded development identifiers and cross-family combinations are
unsupported. Consumers fail closed instead of silently accepting a legacy alias
or coercing one v1 family into another.
