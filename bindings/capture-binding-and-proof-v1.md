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

This table names the existing inputs at each hash stage. Embedded payload bytes
are preserved; only the digest and signing message are canonicalized for hashing.

| Hash or signed input | Still Photo | Live Photo | TAP Video |
| --- | --- | --- | --- |
| `metadataHash` | Exact embedded `manifest.payload` bytes | Same | Same |
| Primary `assetHash` | Complete HEIC/JPEG except the complete proof-slot container; this includes XMP and any auxiliary depth item | Same primary-photo rule; the paired MOV is not part of this hash | Complete MP4 except the complete proof-slot UUID box; this includes the manifest UUID box, RGB/audio media, KLV depth samples, sample tables, and other MP4 bytes |
| `signedResources.primaryPhoto` | Omitted | Same value and excluded range as `assetHash` | Omitted |
| `signedResources.tapDepthManifestPayload` | Omitted | Same hash value as `metadataHash`; also binds exact payload byte count | Omitted |
| `signedResources.pairedLivePhotoVideo` | Omitted | SHA-256 of every byte of the paired MOV | Omitted |
| `contentDigest` canonical JSON | `assetHash`, `metadataHash`, located `proofSlot`, recorded `depthResource`, family/manifest IDs, capture ID, and capture time | All Still fields plus the ordered three-entry `signedResources` array | Same fields as Still, with the recorded video `depthResource` |
| `signingBinding.bodySHA256` | SHA-256 of the complete canonical `contentDigest` above | Same | Same |
| App Attest `clientDataHash` | SHA-256 of the complete canonical `signingBinding` | Same | Same |

`manifest.schema` and `manifest.proofs` are not hashed inside `metadataHash`.
The exact manifest family is still authenticated because
`contentDigest.manifestSchemaID` participates in `bodySHA256`.
Producers write `manifest.proofs` as an empty array. Verification ignores that
member as a proof source; its bytes remain covered by `assetHash`. The fixed proof-slot bytes are
excluded from the primary `assetHash` because they contain the assertion that
is produced from that hash chain; the slot location and length are still bound
through `contentDigest.proofSlot`.

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

The serialized proof-value member is named `contentDigest`.

| Field | JSON type | Presence | Required meaning |
| --- | --- | --- | --- |
| `schemaID` | string | Required | Exact content-binding family from the matrix. |
| `manifestSchemaID` | string | Required | Exact paired manifest family. |
| `captureID` | string | Required | Exact `manifest.payload.id`. |
| `capturedAt` | string | Required | Exact `manifest.payload.capturedAt`. |
| `assetHash` | object | Required | Primary artifact format-native byte-range hash. |
| `metadataHash` | object | Required | Exact embedded manifest payload hash. |
| `proofSlot` | object | Required | Located fixed-slot descriptor. |
| `depthResource` | object | Required | Recorded depth declaration, authenticated as written. |
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

The payload alone is not the `assetHash` excluded range. That range is the
complete enclosing UUID box or JPEG APP11 segment defined by
[Photo Containers v1](../containers/photo-containers-v1.md) or
[TAP Video MP4 Container and Timed Depth v1](../containers/tap-video-container-v1.md).

### `depthResource`

The producer records its auxiliary-depth observation using these descriptors:

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

## Producer signing procedure

1. Finish the primary file, embedded manifest and one empty fixed proof slot;
   finish the MOV too for Live Photo. Locate the slot and manifest using the
   container contract and require the matching manifest/binding family.
2. Build the fields below the hash table from those exact bytes and recorded
   declarations; calculate its stages through the raw 32-byte `clientDataHash`.
3. Call App Attest with that hash and the registered capture key. Encode the
   returned assertion, digest and signing binding in the proof objects above.
4. Fill only the located slot, with the specified envelope and zero padding.
   Do not resize the file, move the slot or rewrite the manifest.
5. Reopen the final bytes and run the local reconstruction below before export.

Key registration and retry follow [BackendContract.md](../BackendContract.md)
and [ProductContract.md](../ProductContract.md#6-credential-and-verification-ux).

## Local reconstruction and cryptographic verification

Verification recomputes the same hash stages, then authenticates the assertion.
It does not decode media or assess sample values, ordering, counts, alignment,
capture settings, calibration, quality or content writing style.

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

1. locate an accepted, active credential for `keyId` and its registered App
   Attest public key, App ID, environment, and backend-owned counter/replay
   state;
2. canonicalize the submitted `signingBinding` with this contract's profile and
   SHA-256 hash those bytes to reconstruct the raw 32-byte `clientDataHash`
   exactly as the producer did;
3. base64url-decode and CBOR-decode `assertionObject`, then extract its
   `authenticatorData` and signature;
4. compute `nonce = SHA-256(authenticatorData || clientDataHash)` and verify the
   assertion signature over that nonce with the public key registered for
   `keyId`;
5. require the authenticator RP ID to match SHA-256 of the registered App ID,
   require the registered environment and credential state to be accepted, and
   apply the backend contract's counter and replay policy; and
6. return a valid result only when every required credential, assertion,
   identity, binding, and backend-policy check succeeds.

Steps 3 through 5 follow Apple's
[server-side assertion validation](https://developer.apple.com/documentation/devicecheck/validating-apps-that-connect-to-your-server)
relationship. This artifact contract fixes the capture-specific client data
and byte relationship; [BackendContract.md](../BackendContract.md) owns counter persistence,
out-of-order submission policy, replay handling, endpoint responses, and audit.
The backend does not receive `contentDigest`, the manifest, the primary media,
depth bytes, or the paired MOV. Its valid result means the registered App
Attest key signed the submitted `signingBinding`. A final artifact verdict is
valid only when that server result is joined with the already-passing local
artifact-binding scope. Capture signing has no server freshness challenge, so
this proves the signed binding, not scene truth or non-replay of an otherwise
valid artifact.
