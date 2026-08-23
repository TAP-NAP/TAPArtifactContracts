# Capture Binding and Proof v1

Tasks: `TAP-0094`, `TAP-0095`. Source boundary:
[SOURCE_SNAPSHOT.md](../SOURCE_SNAPSHOT.md).

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

## Hash participation by artifact family

This table is the normative answer to which bytes and manifest values are
combined at each layer. A check mark means the named input participates in that
layer, directly or through the object named in the row.

| Hash or signed input | Still Photo | Live Photo | TAP Video |
| --- | --- | --- | --- |
| `metadataHash` | Canonical `manifest.payload` only | Canonical `manifest.payload` only | Canonical `manifest.payload` only |
| Primary `assetHash` | Complete HEIC/JPEG except the complete proof-slot container; this includes XMP and any auxiliary depth item | Same primary-photo rule; the paired MOV is not part of this hash | Complete MP4 except the complete proof-slot UUID box; this includes the manifest UUID box, RGB/audio media, KLV depth samples, sample tables, and other MP4 bytes |
| `signedResources.primaryPhoto` | Omitted | Same value and excluded range as `assetHash` | Omitted |
| `signedResources.tapDepthManifestPayload` | Omitted | Same hash value as `metadataHash`; also binds canonical payload byte count | Omitted |
| `signedResources.pairedLivePhotoVideo` | Omitted | SHA-256 of every byte of the paired MOV | Omitted |
| `contentDigest` canonical JSON | `assetHash`, `metadataHash`, located `proofSlot`, actual `depthResource`, family/manifest IDs, capture ID, and capture time | All Still fields plus the ordered three-entry `signedResources` array | `assetHash`, `metadataHash`, located `proofSlot`, manifest-derived `depthResource`, family/manifest IDs, capture ID, and capture time |
| `signingBinding.bodySHA256` | SHA-256 of the complete canonical `contentDigest` above | Same | Same |
| App Attest `clientDataHash` | SHA-256 of the complete canonical `signingBinding` | Same | Same |

`manifest.schema` and `manifest.proofs` are not hashed inside `metadataHash`.
The exact manifest family is still authenticated because
`contentDigest.manifestSchemaID` participates in `bodySHA256`.
`manifest.proofs` MUST be an empty array. The fixed proof-slot bytes are
excluded from the primary `assetHash` because they contain the assertion that
is produced from that hash chain; the slot location and length are still bound
through `contentDigest.proofSlot`.

Photo auxiliary depth participates as bytes already inside the HEIC/JPEG
`assetHash`, not as a separately converted pixel plane. TAP Video KLV depth
samples likewise participate as MP4 bytes inside `assetHash`; their declared
coverage also participates through the manifest payload and `depthResource`.
Only Live Photo has a second file, so only Live Photo needs the full-file MOV
hash in `signedResources`.

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

## Producer signing procedure

The producer MUST perform these operations in order for each artifact. The
procedure is the same for all three families except where a row in the hash
participation table differs.

1. Finish the primary HEIC/JPEG/MP4 container with exactly one empty fixed proof
   slot. Finish the paired MOV first for Live Photo.
2. Locate and validate the actual slot, parse the embedded manifest, require
   `manifest.proofs: []`, and require the exact manifest family for the selected
   artifact route.
3. Carry the exact manifest capture time into the digest. Validate the manifest
   capture ID, output/container facts, depth availability, and—when
   applicable—the Live Photo pairing declaration against the resources being
   signed.
4. Compute `assetHash` and `metadataHash` from the exact inputs in the family
   table. For Live Photo, also compute the ordered three-entry
   `signedResources` array, including the full paired-MOV hash.
5. Assemble the complete family-specific `contentDigest`, including the
   located slot descriptor and actual depth descriptor.
6. Canonicalize `contentDigest`, hash those bytes with SHA-256, and place the
   unpadded base64url digest in `signingBinding.bodySHA256`.
7. Assemble the four-field `signingBinding`, canonicalize it, and SHA-256 hash
   those bytes. Pass the raw 32-byte digest—not its base64url text—to App Attest
   as `clientDataHash` for the already registered capture key.
8. Receive the App Attest assertion bytes. Store their unpadded base64url form
   as `proof.value.assertionObject`, alongside the complete `contentDigest`,
   key handle, and complete `signingBinding`.
9. Canonicalize that proof-value object, encode those bytes as unpadded
   base64url in the outer proof envelope, then canonicalize the envelope.
10. Write only that envelope into the already located fixed slot and leave the
    remaining slot payload zero-filled. The producer MUST NOT resize the file,
    move the slot, or rewrite the manifest while filling the proof.
11. Reopen the final artifact, repeat the applicable local reconstruction and
    relationship checks below, and reject it before export if any byte,
    descriptor, family, ID, or timestamp no longer matches.

App Attest key creation, attestation registration, credential recovery, and
retry policy are producer/backend lifecycle concerns. Their prerequisite here
is only that `keyId` identifies a backend-registered App Attest public key and
the producer can ask the corresponding system-protected private key to generate
the assertion. The private key and raw media bytes never enter this repository.

The backend request contains only `keyId`, `assertionObject`, and the complete
`signingBinding`. The backend verifies registered-key/App Attest semantics; it
does not receive or hash the photo, manifest payload, or MOV. Therefore a local
verifier MUST first recompute and compare the artifact byte binding and MUST NOT
treat a backend-valid response alone as proof about the received media.

## Local reconstruction and cryptographic verification

Verification has two different gates. The local verifier proves that the
received artifact reconstructs the binding carried in the proof. The backend
then performs the cryptographic App Attest assertion verification. Neither gate
substitutes for the other.

### Local artifact-binding gate

The local verifier MUST, in order:

1. Resolve the received primary artifact and optional paired MOV, and name the
   verification scope. Still Photo and TAP Video have one full-artifact scope.
   Live Photo has either a full scope with matching MOV bytes or an explicitly
   limited primary-photo scope. A `.tapnap` sidecar may locate resources but
   MUST NOT supply trusted family, hash, or verdict facts.
2. Bound the input, identify HEIC/JPEG/MP4, locate exactly one supported proof
   slot and manifest, and validate their binary framing before allocating or
   hashing attacker-declared lengths.
3. Parse the envelope and decoded proof value; require the exact proof type,
   algorithm, non-empty `keyID`, non-empty assertion, zero slot padding, empty
   manifest `proofs`, and an allowed manifest/content-binding family pair.
4. Recompute every family-specific input available to the named scope from the
   received bytes and embedded payload; do not copy an available byte-derived
   value from the proof.
5. For Still Photo, TAP Video, and full Live Photo, rebuild the complete
   `contentDigest` and compare it structurally and completely with
   `proof.value.contentDigest`, including byte counts, excluded range, slot
   offsets, depth declaration, ordered Live resources, IDs, and timestamps.
6. For a Live Photo primary-only scope, recompute and compare the primary
   `assetHash`, `metadataHash`, slot, depth, primary resource, and manifest
   resource. Require a complete, structurally valid signed paired-MOV
   descriptor, but do not claim its hash or the complete Live `contentDigest`
   was independently reconstructed without matching MOV bytes.
7. For a full-artifact scope, canonicalize the rebuilt `contentDigest`; for a
   Live Photo primary-only scope, canonicalize the structurally validated
   embedded `contentDigest`. Recompute `bodySHA256`, rebuild the complete
   `signingBinding`, and compare it with `proof.value.signingBinding`.
8. Require the outer `keyID` to equal `proof.value.keyId`, the outer
   `createdAt` to equal `contentDigest.capturedAt`, and every capture ID and
   family relationship to agree.

A full Live Photo result additionally requires the supplied MOV's complete
bytes to match the signed `pairedLivePhotoVideo` descriptor. With absent or
mismatching MOV bytes, a verifier may report only the clearly labelled
primary-photo scope after the limited checks above; it MUST warn about the MOV
state and MUST NOT claim the MOV bytes or full Live Photo were verified. The
backend gate authenticates that the registered key signed the embedded digest,
including its paired-MOV descriptor; it does not turn an unreceived or
mismatching MOV into locally verified bytes.

Passing this local gate authenticates internal byte-binding relationships; it
does not cryptographically verify the App Attest assertion. A consumer may run
separately bounded preview or TAP Video semantic inspection while the backend
gate is pending, but those results remain untrusted and cannot produce a final
authenticated verdict.

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
and byte relationship; the producer/backend contract owns counter persistence,
out-of-order submission policy, replay handling, endpoint responses, and audit.
It therefore does not invent a stricter counter rule here.

The backend does not receive `contentDigest`, the manifest, the primary media,
depth bytes, or the paired MOV. Its valid result means the registered App
Attest key signed the submitted `signingBinding`. A final artifact verdict is
valid only when that server result is joined with the already-passing local
artifact-binding scope. Capture signing has no server freshness challenge, so
this proves the signed binding, not scene truth or non-replay of an otherwise
valid artifact.

## Fail-closed rejection summary

Missing slots, duplicate slots, malformed lengths, invalid magic/version,
non-zero trailing padding, non-empty manifest proofs, mismatched family pairs,
missing Live Photo resources, hash mismatches, or relationship mismatches MUST
fail the affected verification scope before the backend request. An unknown or
inactive key, malformed assertion, signature failure, app/environment mismatch,
or backend counter/replay-policy failure MUST fail at the backend gate. A
transport label, decoded-pixel match, or backend-valid result alone MUST NOT
upgrade either failure to valid.

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
