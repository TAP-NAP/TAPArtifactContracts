# Capture Binding and Proof v1

This document defines the shared Still Photo, Live Photo, and TAP Video byte-
binding and App Attest proof relationship. Key registration and server trust are
defined in [BackendContract.md](../BackendContract.md); credential lifecycle and
product verification copy are defined in [ProductContract.md](../ProductContract.md#6-credential-and-verification-ux).

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

Select the algorithm by the exact manifest `schema.id` and its matching binding
family. The manifest's descriptive `schema.version` and `schema.mediaType` do
not select it; the payload hash media type is the fixed value in this matrix.

## Hash participation by artifact family

The fields below hash these original blobs. Only `contentDigest` and
`signingBinding` are canonicalized for hashing.

| Blob | Still / Live Photo | TAP Video |
| --- | --- | --- |
| [`assetHash`](#assethash) | HEIC/JPEG, including XMP and auxiliary depth; excludes the complete proof-slot container | MP4, including manifest, RGB/audio, KLV depth, sample tables and other bytes; excludes the complete proof-slot UUID box |
| [`metadataHash`](#metadatahash) | Exact embedded `manifest.payload` JSON bytes | Same |
| [Paired MOV](#live-photo-signedresources) | Live only: complete MOV, separate from the primary asset | Omitted |

`metadataHash` excludes `manifest.schema` and `manifest.proofs`; the manifest
family is bound by `contentDigest.manifestSchemaID`. Producers write
`manifest.proofs` as an empty array. Readers ignore it as a proof source, but
its bytes remain in `assetHash`. The fixed proof slot is excluded to avoid
hashing the assertion into its own input; `contentDigest.proofSlot` still binds
its location and length.

## TAP capture canonical JSON

**TAP capture canonical JSON** is the unchanged encoding for `contentDigest`
and `signingBinding`. Producers also use this style for content JSON. Content
style is not a signature-acceptance condition. The encoding rules are:

- there is no byte-order mark, insignificant whitespace, or trailing data;
- object member names are unique and sorted recursively in ascending unsigned
  UTF-8 byte order; the defined v1 field names are ASCII;
- array order is preserved and significant;
- `null`, `true`, and `false` use those lowercase spellings;
- values in fields declared `integer` use the shortest base-10 form, with no
  leading zero and no positive sign; integer zero is `0` and a negative integer
  begins with `-`;
- `"`, `\\`, backspace, tab, newline, form feed, and carriage return are escaped
  as `\"`, `\\`, `\b`, `\t`, `\n`, `\f`, and `\r`; other U+0000 through
  U+001F scalars use lowercase `\u00xx`; `/` and all other Unicode scalars are
  emitted directly as UTF-8, with no Unicode normalization;
- absent optional members are omitted except where a manifest table explicitly
  requires `null`; and
- values in fields declared `number` use valid finite JSON number tokens;
  floating-point negative zero may therefore appear as `-0`. NaN and infinities
  are forbidden, and no consumer may rewrite a number token before hashing.

For `metadataHash` and the Live payload byte count, locate the exact raw
`manifest.payload` value after decoding its container wrapper. Never reserialize
it: whitespace, member order and number spelling remain part of those bytes.
The historical `canonical-json` kind and media-type identifiers stay unchanged.

This profile is not RFC 8785/JCS. V1 has no general binary32/binary64 decimal
formatter beyond the [exact vector](../examples/vectors/tap-capture-canonical-json-v1.json).
The digest, signing binding and proof contain no floating-point fields; their
canonical encoding is fully specified above.

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

All members of `contentDigest` are required except `signedResources`, which is
required for Live Photo and omitted for Still/Video.

| Field | JSON type | Meaning |
| --- | --- | --- |
| `schemaID` | string | Exact content-binding family from the matrix. |
| `manifestSchemaID` | string | Exact paired manifest family. |
| `captureID` | string | Exact `manifest.payload.id`. |
| `capturedAt` | string | Exact `manifest.payload.capturedAt`. |
| `assetHash` | object | Primary artifact byte-range hash. |
| `metadataHash` | object | Embedded manifest payload hash. |
| `proofSlot` | object | Located fixed-slot descriptor. |
| `depthResource` | object | Recorded depth declaration, authenticated as written. |
| `signedResources` | array | Three Live Photo descriptors in the order defined below. |

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
marker and length. See [Photo Containers v1](../containers/photo-containers-v1.md)
and [TAP Video MP4 Container and Timed Depth v1](../containers/tap-video-container-v1.md).

### `metadataHash`

| Field | JSON type | Required value or derivation |
| --- | --- | --- |
| `kind` | string | `canonical-json` |
| `mediaType` | string | Family-specific payload media type from the matrix |
| `algorithm` | string | `SHA-256` |
| `value` | string | Base64url SHA-256 of exact embedded `manifest.payload` JSON bytes |

### `proofSlot`

| Field | JSON type | Required meaning or value |
| --- | --- | --- |
| `kind` | string enum | `bmff-uuid-proof-slot` for HEIC or TAP Video MP4; `jpeg-app11-proof-slot` for JPEG |
| `offset` | integer | Start of the complete slot container range in the primary artifact |
| `length` | integer | Complete slot container length |
| `payloadOffset` | integer | Start of the 61,440-byte TAP slot payload |
| `payloadLength` | integer | Exact value `61440` |
| `padding` | string | `zero-filled-after-envelope` |

The values MUST equal the slot actually located in the received file. Every
HEIC, JPEG, and TAP Video slot carries this same 61,440-byte payload:

| Payload-relative byte range | Size | Encoding and meaning |
| --- | ---: | --- |
| `0..<20` | 20 | ASCII `TAPCAM-PROOF-SLOT-V1` |
| `20..<24` | 4 | Producer-reserved zero bytes |
| `24..<28` | 4 | Unsigned 32-bit big-endian version, exact value `1` |
| `28..<32` | 4 | Unsigned 32-bit big-endian proof-envelope byte length `N` |
| `32..<(32+N)` | `N` | Canonical proof-envelope JSON bytes |
| `(32+N)..<61440` | remainder | Zero padding |

The maximum envelope length is `61408` bytes. An unsigned pending artifact has
`N == 0`; a signed artifact requires `N > 0`. The producer MUST zero the four
reserved bytes and every byte after the envelope. Non-zero reserved bytes are
producer nonconformance; a v1 reader MAY reject them but MUST NOT assign them
meaning. A reader MUST reject an incorrect payload size, magic, version,
envelope length or JSON, or non-zero trailing padding, as well as a missing or
duplicate slot.

### `depthResource`

The producer records its auxiliary-depth observation using these descriptors:

| Depth availability | `presence` | `binding` | `interpretation` | `platformPresenceCheck` |
| --- | --- | --- | --- | --- |
| `available` | `required` | `covered-by-assetHash` | `not-part-of-base-signature` | `AVDepthData-readback` |
| `unavailable` | `unavailable` | `not-present` | `no-depth-captured` | `AVDepthData-readback-missing` |

All four members are required strings. Auxiliary photo depth is covered as
primary-file bytes, with no separately hashed converted plane. Preserve
`not-part-of-base-signature` exactly: it does not exclude auxiliary container
bytes from `assetHash`.

For TAP Video, `depthResource` is derived from the signed manifest's
`depthCoverage.sampleCount`:

| Video depth coverage | `presence` | `binding` | `interpretation` | `platformPresenceCheck` |
| --- | --- | --- | --- | --- |
| `sampleCount > 0` | `captured` | `covered-by-assetHash` | `not-part-of-base-signature` | `TAPVideoManifest.depthCoverage` |
| `sampleCount == 0` | `no-samples` | `coverage-recorded-in-manifest` | `not-part-of-base-signature` | `TAPVideoManifest.depthCoverage` |

Verification preserves this declaration in `contentDigest`; the complete digest
hash authenticates it. Auxiliary-depth detection is not repeated.

### Live Photo `signedResources`

The Live Photo array MUST contain exactly these three objects in this producer
order. Every object has required string `role`, `kind`, `mediaType`, `algorithm`,
`value`, and `binding`, plus required integer `byteCount`. `excludedRanges` is
present only where shown and is omitted—not `null`—otherwise.

| Role | Required descriptor |
| --- | --- |
| `primaryPhoto` | `kind`, `algorithm`, `byteCount`, `value`, and `excludedRanges` equal `assetHash`; `mediaType` is `public.heic` or `public.jpeg`; `binding` is `format-native-byte-ranges`. |
| `tapDepthManifestPayload` | `kind: canonical-json`; Live payload media type; `algorithm: SHA-256`; `byteCount` is exact embedded payload byte length; `value` equals `metadataHash.value`; `binding: canonical-json`; `excludedRanges` omitted. |
| `pairedLivePhotoVideo` | `kind: format-native-full-file`; `mediaType: com.apple.quicktime-movie`; `algorithm: SHA-256`; full MOV byte count; base64url hash of the complete MOV; `binding: full-file`; `excludedRanges` omitted. |

The MOV digest MUST NOT appear in `manifest.payload.livePhoto`.

## Proof envelope JSON

The fixed slot's envelope bytes are canonical JSON for this object:

| Field | JSON type | Presence | Required meaning or value |
| --- | --- | --- | --- |
| `type` | string | Required | `appAttestAssertion` |
| `algorithm` | string | Required | `TAPCam.AppAttestCaptureSignature.v1` |
| `keyID` | string | Required | Non-empty App Attest key handle |
| `createdAt` | string | Producer-required | Producer writes `contentDigest.capturedAt`; verification ignores this excluded-envelope copy. |
| `value` | string | Required | Unpadded base64url of canonical `CaptureAssertionProofValue` JSON bytes |

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

`clientDataHash = SHA-256(canonical(signingBinding))`, where `canonical` is
[TAP capture canonical JSON](#tap-capture-canonical-json). Pass the raw 32-byte
hash to App Attest, not its base64url text.

## Producer signing procedure

1. Finish the primary file, embedded manifest and one empty fixed proof slot;
   finish the MOV too for Live Photo. Locate the slot and manifest using the
   container contract and require the matching manifest/binding family.
2. Build `contentDigest` from those bytes and recorded declarations, then
   `signingBinding` and `clientDataHash` as defined above.
3. Call App Attest with that hash and the registered capture key. Encode the
   returned assertion, digest and signing binding in the proof objects above.
4. Fill only the located slot, with the specified envelope and zero padding.
   Do not resize the file, move the slot or rewrite the manifest.
5. Reopen the final bytes and run the local reconstruction below before export.

Key registration and retry follow [BackendContract.md](../BackendContract.md)
and [ProductContract.md](../ProductContract.md#6-credential-and-verification-ux).

## Local reconstruction and cryptographic verification

Verification recomputes the hash stages, then authenticates the assertion.
Media decoding, sample values/order/counts/alignment, capture settings,
calibration, quality and content writing style do not gate signature acceptance.

### Local artifact-binding gate

1. Resolve the primary and optional MOV. Bound reads, locate exactly one
   manifest and proof slot, and check their container and slot framing.
2. Parse the proof objects against the field tables above, including the exact
   proof type/algorithm, non-empty key/assertion, and matching signing families.
3. Recompute the available blob hashes, byte counts, slot/range and manifest
   identity fields. Preserve the embedded `depthResource`; compare the rebuilt
   fields with `contentDigest`, including Live resource descriptors.
4. Recompute `bodySHA256` and `signingBinding` and compare both. Require matching
   outer/inner key IDs and capture IDs. Display only the bound capture time.

Still and Video require complete reconstruction. Live Photo uses these scopes:

| Received MOV | Reconstructed fields | Result after backend verification |
| --- | --- | --- |
| Exact signed bytes, including an empty file if its hash matches | Complete `contentDigest` | Full Live Photo |
| Missing or mismatching | Primary asset, payload, slot and their two signed resources; validate the full signed MOV descriptor without claiming its bytes match | Primary photo only, with an explicit missing/mismatch warning |

For primary-only Live verification, use the complete embedded digest to
recompute `bodySHA256`. Its signature authenticates the MOV declaration, not
unreceived or mismatching MOV bytes. A malformed signed resource descriptor
fails verification. Local success alone is not App Attest verification.

### Backend App Attest gate

Only after the local gate passes does the verifier submit the unchanged
`keyId`, `assertionObject`, and `signingBinding`. The backend MUST:

1. locate the credential for `keyId`, its registered App Attest public key,
   App ID, environment and backend-owned counter/replay state;
2. reconstruct `clientDataHash` as defined above;
3. base64url-decode and CBOR-decode `assertionObject`, then extract its
   `authenticatorData` and signature;
4. compute `nonce = SHA-256(authenticatorData || clientDataHash)` and verify the
   assertion signature over that nonce with the registered public key;
5. require the authenticator RP ID to match SHA-256 of the registered App ID,
   an accepted registered environment and an accepted, active credential, and
   apply the backend contract's counter and replay policy; and
6. return a valid result only when every required credential, assertion,
   identity, binding, and backend-policy check succeeds.

Steps 3–5 follow Apple's
[server-side assertion validation](https://developer.apple.com/documentation/devicecheck/validating-apps-that-connect-to-your-server).
[BackendContract.md](../BackendContract.md) owns counter persistence,
out-of-order submission, replay, endpoint responses and audit.
The backend does not receive `contentDigest`, the manifest, the primary media,
depth bytes, or the paired MOV. Its valid result means the registered App
Attest key signed the submitted `signingBinding`. A final artifact verdict is
valid only when that server result is joined with the already-passing local
artifact-binding scope. Capture signing has no server freshness challenge, so
this proves the signed binding, not scene truth or non-replay of an otherwise
valid artifact.
