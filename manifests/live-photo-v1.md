# Live Photo Manifest v1

This is a distinct manifest family for one TAP Live Photo: a primary HEIC/JPEG
photo containing the manifest, auxiliary still-photo depth when available, and
the proof slot, plus one byte-preserved paired MOV. It is not a Still Photo alias
and it does not claim per-frame MOV depth.

The accepted synthetic shape and its expected decision are indexed in
[`examples/README.md`](../examples/README.md); it is not a complete signed-media
or canonical-byte golden vector.

## Family identity

The following values MUST match exactly.

| Field | Required JSON value |
| --- | --- |
| `schema.id` | `urn:tapnap:tapcam:live-photo-manifest:v1` |
| `schema.version` | integer `1` |
| `schema.mediaType` | `application/vnd.tapnap.live-photo-manifest+json;version=1` |
| `schema.xmpNamespaceURI` | `urn:tapnap:tapcam:depth:1.0` |
| `schema.xmpPrefix` | `tapdepth` |
| `schema.xmpManifestPath` | `tapdepth:Manifest` |
| Canonical payload media type | `application/vnd.tapnap.live-photo-manifest.payload+json;version=1` |

The top-level members `schema`, `payload`, and `proofs` are required.
`proofs` MUST be `[]`; the actual proof remains in the primary photo's fixed
proof slot.

## Payload composition

Live Photo v1 uses every common photo-payload field, type, unit, coordinate
system, enum vocabulary, omission rule, and privacy boundary defined in
[Still Photo Manifest v1](still-photo-v1.md#payload-root), with two normative
differences only:

1. the exact Live Photo `schema` and canonical payload media type above replace
   their Still Photo counterparts; and
2. `payload.livePhoto` is required and has the object below rather than being
   omitted.

This structural reuse does not merge the two families. A consumer MUST route on
the complete manifest and content-binding family pair and MUST reject a Live
Photo payload under the Still Photo family or a Still payload under the Live
Photo family.

## `payload.livePhoto`

| Field | JSON type | Presence | Meaning, unit, vocabulary |
| --- | --- | --- | --- |
| `presence` | string enum | Required | Exact value `paired-video`. |
| `pairedVideoFilename` | string | Required | Exact value `paired-video.mov`. This names the required logical resource; it is not a hash. |
| `durationSeconds` | number | Required | Non-negative MOV duration in seconds, clamped to at least zero by the producer. |
| `photoDisplayTimeSeconds` | number | Required | Non-negative still-photo display time within the MOV timeline, in seconds, clamped to at least zero. |
| `width` | integer | Required | MOV encoded width in pixels, sourced from resolved Live Photo movie dimensions. |
| `height` | integer | Required | MOV encoded height in pixels. |
| `videoCodec` | string | Optional, omitted | Selected `AVVideoCodecType.rawValue`, normally `hvc1`, then `avc1`, then another platform-supported value; omitted if no codec value was selected. |
| `audio` | string enum | Required | `captured` or `not-captured`. This is the capture-request/result fact; it is not an audio-track decoder result. |

## Paired-resource binding

`payload.livePhoto` describes the MOV but MUST NOT contain its digest. The Live
Photo content-binding family, exact ordered resource descriptors, and full-file
MOV hash are defined once under
[Live Photo `signedResources`](../bindings/capture-binding-and-proof-v1.md#live-photo-signedresources).
The primary photo remains the only depth resource. Nothing in this family
describes or binds per-frame MOV depth.

## Input and failure rules

- A complete Live Photo verification input MUST contain one primary photo and
  the MOV named by `pairedVideoFilename`.
- The Apple Photos resource pairing is `.photo` plus `.pairedVideo`; package
  filenames are routing inputs only and never replace the embedded hash chain.
- If capture requested Live Photo but no movie complement was produced, TAPCam
  emits the Still Photo manifest and content-binding families instead. It MUST
  NOT emit a partial Live Photo manifest.
- If a verifier receives this family without the MOV, it may report the bound
  primary-photo checks separately, but it MUST report the complete Live Photo
  resource set as missing/incomplete.
- A MOV hash mismatch fails the Live Photo paired-resource check even when the
  primary photo still matches its signed bytes.
- Presentation or adjusted Photos resources are not substitutes for the
  original `.photo` and `.pairedVideo` bytes.

## Required consistency checks

A conforming verifier MUST:

1. require the exact Live Photo family object, non-null `payload.livePhoto`,
   exact `presence`, and exact `pairedVideoFilename`;
2. require `proofs: []`;
3. apply every common Still/Live photo payload check, including depth
   availability and auxiliary-data presence;
4. require the Live Photo content-binding family and all three named resource
   descriptors; and
5. apply the scope-aware local reconstruction and backend gates in
   [Capture Binding and Proof v1](../bindings/capture-binding-and-proof-v1.md).
