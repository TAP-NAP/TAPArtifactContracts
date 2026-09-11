# Live Photo Manifest v1

This is a distinct manifest family for one TAP Live Photo: a primary HEIC/JPEG
photo containing the manifest, auxiliary still-photo depth when available, and
the proof slot, plus one byte-preserved paired MOV. It is not a Still Photo alias
and it does not claim per-frame MOV depth.

See the [synthetic example](../examples/manifests/live-photo-v1.json).
[Capture Binding and Proof](../bindings/capture-binding-and-proof-v1.md)
owns the signed resources and complete/primary-only verification scopes.

## Family identity

Live Photo inherits the [Still Photo schema](still-photo-v1.md#family-identity),
replacing only these two values:

| Field | Required JSON value |
| --- | --- |
| `schema.id` | `urn:tapnap:tapcam:live-photo-manifest:v1` |
| `schema.mediaType` | `application/vnd.tapnap.live-photo-manifest+json;version=1` |

Producers write top-level `schema`, `payload`, and empty `proofs`.

## Payload composition

Live Photo v1 uses the common photo-payload field descriptions, units, coordinate
conventions, and privacy boundary defined in
[Still Photo Manifest v1](still-photo-v1.md#payload-root), with two normative
differences only:

1. the Live Photo `schema` above replaces its Still Photo counterpart; and
2. `payload.livePhoto` is required and has the object below rather than being
   omitted.

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

If capture requested Live Photo but produced no movie complement, the producer
emits the Still Photo family. The Live family describes the original primary
and MOV, not adjusted Photos resources.
