# Capture Binding and Proof v1

Task: `TAP-0094`. Source boundary: [SOURCE_SNAPSHOT.md](../SOURCE_SNAPSHOT.md).

This document defines the shared Still Photo, Live Photo, and TAP Video byte-
binding and App Attest proof relationship. It does not define key registration,
credential lifecycle, backend deployment, or product verification copy.

## Family matrix

| Artifact | Manifest family | Content-binding family | Payload media type |
| --- | --- | --- | --- |
| Still Photo | `urn:tapnap:tapcam:still-photo-manifest:v1` | `urn:tapnap:tapcam:still-photo-content-binding:v1` | `application/vnd.tapnap.still-photo-manifest.payload+json;version=1` |
| Live Photo | `urn:tapnap:tapcam:live-photo-manifest:v1` | `urn:tapnap:tapcam:live-photo-content-binding:v1` | `application/vnd.tapnap.live-photo-manifest.payload+json;version=1` |
| TAP Video | `urn:tapnap:tapcam:video-manifest:v1` | `urn:tapnap:tapcam:video-content-binding:v1` | `application/vnd.tapnap.video-manifest.payload+json;version=1` |

All three use the distinct signing-binding family
`urn:tapnap:tapcam:app-attest-capture-signing:v1`, operation
`tapcam.capture.sign`, proof type `appAttestAssertion`, and proof algorithm
`TAPCam.AppAttestCaptureSignature.v1`. A shared signing-binding family does not
make the manifest or content-binding families interchangeable.

## End-to-end byte chain

```text
embedded manifest.payload
  -> TAP capture canonical JSON UTF-8 bytes
  -> SHA-256 -> metadataHash.value

primary HEIC/JPEG or TAP Video MP4 bytes minus the entire proof-slot container range
  -> SHA-256 -> assetHash.value

Live Photo only: complete paired-video.mov bytes
  -> SHA-256 -> signedResources.pairedLivePhotoVideo.value

assetHash + metadataHash + proofSlot + depthResource [+ signedResources]
  -> contentDigest object
  -> canonical JSON -> SHA-256 -> signingBinding.bodySHA256

signingBinding object
  -> canonical JSON -> SHA-256 raw 32-byte clientDataHash
  -> App Attest generateAssertion
  -> assertionObject

contentDigest + keyId + assertionObject + signingBinding
  -> canonical proof-value JSON -> base64url -> proof.value
  -> canonical proof JSON -> fixed proof-slot envelope bytes
```

Changing proof-slot contents does not change `assetHash`, because the entire
slot container range—not only the envelope bytes—is excluded.

## TAP capture canonical JSON

Every canonical JSON operation above uses the current Swift
`JSONEncoder` with both `.sortedKeys` and `.withoutEscapingSlashes`:

- output is compact UTF-8 JSON with no pretty-print whitespace;
- object keys are sorted by the encoder at every object level;
- array order is preserved and is significant;
- `/` is not escaped only because of `.withoutEscapingSlashes`;
- optionals are omitted when absent except the manifest's explicit
  `payload.location: null` rule;
- non-finite floating-point values are not encoded; and
- no Unicode normalization or application-level number rewriting is performed.

This profile is called **TAP capture canonical JSON**. It is not claimed to be
RFC 8785/JCS. A non-Swift verifier MUST produce identical bytes for the values
written by the current producer before comparing hashes. The formatted JSON
examples in this repository are shape examples, not canonical-byte vectors.

## Hash and encoding primitives

- Every digest uses SHA-256 over the exact byte sequence named here.
- Every hash string is unpadded base64url: standard Base64 with `+` changed to
  `-`, `/` changed to `_`, and all trailing `=` removed.
- `byteCount`, offsets, lengths, and payload lengths are decimal byte counts from
  the beginning of the containing file; they do not count decoded pixels or
  characters.
- A verifier MUST NOT use decoded image pixels, browser canvas pixels, decoded
  MOV frames, or converted metric-depth samples as a base binding input.

## `contentDigest`

The proof-value member is named `contentDigest` in serialized JSON even though
the current Swift type is also called `CaptureContentBinding`.

| Field | JSON type | Presence | Required meaning |
| --- | --- | --- | --- |
| `schemaID` | string | Required | Exact content-binding family from the matrix. |
| `manifestSchemaID` | string | Required | Exact paired manifest family. |
| `captureID` | string | Required | Exact `manifest.payload.id`. |
| `capturedAt` | string | Required | Exact `manifest.payload.capturedAt`. |
| `assetHash` | object | Required | Primary photo format-native byte-range hash. |
| `metadataHash` | object | Required | Canonical manifest payload hash. |
| `proofSlot` | object | Required | Located fixed-slot descriptor. |
| `depthResource` | object | Required | Actual auxiliary-depth presence/binding declaration. |
| `signedResources` | array | Still/Video: omitted; Live: required | Exactly three Live Photo resource descriptors in producer order. |

### `assetHash`

| Field | JSON type | Required value or derivation |
| --- | --- | --- |
| `kind` | string | `c2pa-style-format-native-byte-ranges` |
| `fileContainer` | string enum | `heic`, `jpeg`, or `mp4`, matching actual input bytes |
| `algorithm` | string | `SHA-256` |
| `byteCount` | integer | Full primary artifact file length, including the proof slot |
| `value` | string | Base64url SHA-256 of `file[0..<slot.offset] || file[slot.offset+slot.length..<file.count]` |
| `excludedRanges` | array | Exactly one `ExcludedRange`, describing the entire proof-slot container range |

The sole `ExcludedRange` has required integer `offset`, required integer
`length`, and required string `reason: "tap-proof-slot"`. For HEIC or TAP Video
MP4 it covers the complete top-level UUID box, including its BMFF header, user
type, and payload. For JPEG it covers the complete APP11 segment, including
marker and length.

### `metadataHash`

| Field | JSON type | Required value or derivation |
| --- | --- | --- |
| `kind` | string | `canonical-json` |
| `mediaType` | string | Family-specific payload media type from the matrix |
| `algorithm` | string | `SHA-256` |
| `value` | string | Base64url SHA-256 of canonical `manifest.payload` JSON bytes |

`manifest.schema` and `manifest.proofs` are not part of `metadataHash`; family
identity is separately bound through `manifestSchemaID`, and `manifest.proofs`
MUST remain empty.

### `proofSlot`

| Field | JSON type | Required meaning or value |
| --- | --- | --- |
| `kind` | string enum | `bmff-uuid-proof-slot` for HEIC or TAP Video MP4; `jpeg-app11-proof-slot` for JPEG |
| `offset` | integer | Start of the complete slot container range in the primary artifact |
| `length` | integer | Complete slot container length |
| `payloadOffset` | integer | Start of the 61,440-byte TAP slot payload |
| `payloadLength` | integer | Exact value `61440` |
| `padding` | string | `zero-filled-after-envelope` |

The values MUST equal the slot actually located in the received file. Binary
layout is defined in
[Photo Containers v1](../containers/photo-containers-v1.md#fixed-proof-slot-payload).

### `depthResource`

The descriptor is selected from actual auxiliary-depth readback and must agree
with both manifest availability fields.

| Depth availability | `presence` | `binding` | `interpretation` | `platformPresenceCheck` |
| --- | --- | --- | --- | --- |
| `available` | `required` | `covered-by-assetHash` | `not-part-of-base-signature` | `AVDepthData-readback` |
| `unavailable` | `unavailable` | `not-present` | `no-depth-captured` | `AVDepthData-readback-missing` |

All four members are required strings. When available, auxiliary photo-depth bytes
are covered transitively as bytes inside the primary photo's `assetHash`; there
is no separately hashed converted depth plane. The historical string
`not-part-of-base-signature` MUST be preserved exactly and must not be
misinterpreted as excluding the auxiliary container bytes from `assetHash`.

For TAP Video, `depthResource` is derived from the signed manifest's
`depthCoverage.sampleCount`:

| Video depth coverage | `presence` | `binding` | `interpretation` | `platformPresenceCheck` |
| --- | --- | --- | --- | --- |
| `sampleCount > 0` | `captured` | `covered-by-assetHash` | `not-part-of-base-signature` | `TAPVideoManifest.depthCoverage` |
| `sampleCount == 0` | `no-samples` | `coverage-recorded-in-manifest` | `not-part-of-base-signature` | `TAPVideoManifest.depthCoverage` |

Stored KLV samples are already bytes inside the MP4 `assetHash`. A zero-depth
TAP Video remains a valid family member and binds the signed no-samples fact in
its manifest payload.

### Live Photo `signedResources`

The Live Photo array MUST contain exactly these three objects in this producer
order. Every object has required string `role`, `kind`, `mediaType`, `algorithm`,
`value`, and `binding`, plus required integer `byteCount`. `excludedRanges` is
present only where shown and is omitted—not `null`—otherwise.

| Role | Required descriptor |
| --- | --- |
| `primaryPhoto` | `kind`, `algorithm`, `byteCount`, `value`, and `excludedRanges` equal `assetHash`; `mediaType` is `public.heic` or `public.jpeg`; `binding` is `format-native-byte-ranges`. |
| `tapDepthManifestPayload` | `kind: canonical-json`; Live payload media type; `algorithm: SHA-256`; `byteCount` is canonical payload byte length; `value` equals `metadataHash.value`; `binding: canonical-json`; `excludedRanges` omitted. |
| `pairedLivePhotoVideo` | `kind: format-native-full-file`; `mediaType: com.apple.quicktime-movie`; `algorithm: SHA-256`; full MOV byte count; base64url hash of the complete MOV; `binding: full-file`; `excludedRanges` omitted. |

The MOV digest MUST NOT appear in `manifest.payload.livePhoto`.

## Proof envelope JSON

The fixed slot's envelope bytes are canonical JSON for this object:

| Field | JSON type | Presence | Required meaning or value |
| --- | --- | --- | --- |
| `type` | string | Required | `appAttestAssertion` |
| `algorithm` | string | Required | `TAPCam.AppAttestCaptureSignature.v1` |
| `keyID` | string | Required | Non-empty App Attest key handle |
| `createdAt` | string | Required | Exact `contentDigest.capturedAt` |
| `value` | string | Required | Unpadded base64url of canonical `CaptureAssertionProofValue` JSON bytes |

Although the Swift schema types the final three members as optional, a valid
signed artifact requires all of them; omission or `null` is rejected.
`manifest.proofs` remains empty before and after this object is written into the
slot.

After base64url decoding, `proof.value` is this required object:

| Field | JSON type | Required relationship |
| --- | --- | --- |
| `contentDigest` | object | Complete family-specific content digest defined above |
| `keyId` | string | Exact match for outer `proof.keyID` (note the different serialized capitalization) |
| `assertionObject` | string | Non-empty unpadded base64url App Attest assertion bytes |
| `signingBinding` | object | Complete object below |

## `signingBinding` and App Attest input

| Field | JSON type | Required value or derivation |
| --- | --- | --- |
| `bodySHA256` | string | Base64url SHA-256 of canonical `contentDigest` JSON bytes |
| `captureID` | string | Exact `contentDigest.captureID` |
| `operation` | string | `tapcam.capture.sign` |
| `schemaID` | string | `urn:tapnap:tapcam:app-attest-capture-signing:v1` |

The producer canonicalizes this four-member object, SHA-256 hashes those bytes,
and passes the resulting raw 32 bytes as App Attest `clientDataHash`. The
returned assertion bytes become `proof.value.assertionObject`.

The backend request contains only `keyId`, `assertionObject`, and the complete
`signingBinding`. The backend verifies registered-key/App Attest semantics; it
does not receive or hash the photo, manifest payload, or MOV. Therefore a local
verifier MUST first recompute and compare the artifact byte binding and MUST NOT
treat a backend-valid response alone as proof about the received media.

## Fail-closed verification order

A verifier MUST:

1. identify HEIC, JPEG, or TAP Video MP4 and locate exactly one well-formed
   proof slot;
2. read its non-empty, bounded envelope and reject non-zero bytes after it;
3. parse exactly one XMP `tapdepth:Manifest` for a photo or one TAP Video
   manifest UUID box for MP4, require `proofs: []`, and route the exact
   manifest/content-binding family pair;
4. recompute `assetHash`, canonical payload bytes, `metadataHash`, `proofSlot`,
   `depthResource`, and Live Photo resources when applicable;
5. compare the complete reconstructed `contentDigest`, not only its hash values;
6. recompute `signingBinding.bodySHA256` and the complete expected
   `signingBinding`;
7. require proof key, timestamp, capture-ID, and family relationships; and
8. only then submit the unchanged backend request shape when backend assertion
   authenticity is required.

Missing slots, duplicate slots, malformed lengths, invalid magic/version,
non-zero trailing padding, non-empty manifest proofs, mismatched family pairs,
missing Live Photo resources, hash mismatches, or relationship mismatches MUST
fail the affected verification scope.

## Extraction notes

- The current producer supports no-depth Still and Live photos by emitting the
  `unavailable` `depthResource` row above. The current TAPCamVerifier Rust
  reconstruction observed in the extraction snapshot still constructs only the
  `available` row. This producer/consumer discrepancy is recorded rather than
  resolved here; the Product Contract and producer remain the authority for the
  current no-depth capability until a separately approved verifier correction
  is completed.
- Current producer and verifier canonicalization agree for exercised fixtures,
  but this extraction found no complete Still Photo or Live Photo canonical
  golden-byte vector covering high-precision numbers, Unicode, omission, full
  content digest, and signing binding. This document records the current encoder
  profile; adding immutable cross-language vectors is still needed before this
  repository can prove all edge-byte behavior independently.
- The current verifier base64url decoder accepts padded input as a permissive
  decode path. The producer emits unpadded base64url; permissive input acceptance
  is not a producer wire requirement.
