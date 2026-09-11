# `.tapnap` Verification Transport v1

Status: current pre-release v1 transport contract for Still Photo, Live Photo, and TAP Video
Routing sidecar family: `urn:tapnap:tapcam:verification-export:v1`

`.tapnap` is a byte-preserving transport wrapper. It carries original media to
a verifier; it is not an artifact-manifest family, a signature format, or
authenticity evidence. The authoritative manifest and proof remain embedded in
the original media: the primary HEIC/JPEG for Still/Live Photo, or the MP4 for
TAP Video.

A TAP Video package carries one complete original signed MP4, including its
manifest, proof slot, RGB/audio tracks and any depth or telemetry. Producers
MUST NOT extract a second manifest, proof or depth file. Raw MP4 sharing remains
supported. Video support belongs to the current
[pre-release v1 definition](../VERSIONING.md#current-pre-release-policy), not
necessarily earlier producer or verifier revisions.

## Transport identity

| Identifier | V1 value |
| --- | --- |
| Preferred filename | `TAPNAP-Capture.tapnap` |
| Filename extension | `.tapnap` |
| UTI | `net.tapnap.capture-package` |
| MIME type | `application/vnd.tapnap.capture-package+zip` |
| Container | ZIP-compatible archive |

A verifier recognizes a capture package only from the `.tapnap` extension or
the exact TAPNAP MIME type. Generic `.zip`, `application/zip`, ZIP magic, a
legacy marker, or archive contents MUST NOT silently reclassify an input as a
v1 TAPNAP package. Raw HEIC/HEIF/JPG/JPEG and TAP Video MP4 inputs remain
their own input routes.

A v1 producer MUST preserve each media resource byte-for-byte. ZIP compression
method is transport-local and has no signed meaning. A verifier MUST NOT
transcode or normalize declared media before embedded-proof and resource-hash
verification.

## V1 archive layouts

Every package has the root sidecar `tapcam-export.json`. Producers write these
media resources as root entries:

| Artifact | Primary entry | Paired entry |
| --- | --- | --- |
| Still Photo | `primary-photo.heic` or `primary-photo.jpg` | Omitted |
| Live Photo | `primary-photo.heic` or `primary-photo.jpg` | `paired-video.mov` |
| TAP Video | `original-video.mp4` | Omitted |

## `tapcam-export.json`

The sidecar is unsigned routing metadata. Its only authority is to map a small
set of resource roles to exact archive entries. A verifier derives the actual
manifest family and verification scope from the proof-bearing original media,
never from `packageKind`, `mediaType`, warnings, filenames, or archive shape.
The unique primary resource role selects the photo or video parser. The embedded
artifact must then satisfy its own manifest/binding family and byte-hash checks.

Producers write all these fields:

| Field | Type | Required value / meaning |
| --- | --- | --- |
| `schemaID` | string | `urn:tapnap:tapcam:verification-export:v1` |
| `version` | integer | `1` |
| `packageKind` | string | `stillPhoto`, `livePhotoPackage`, or `tapVideo` |
| `resources` | array | Role-to-entry mappings defined below. |
| `warningLabels` | array of strings | Labels for presentation-adjustment resources detected by the producer; empty when none. |
| `warnings` | array of strings | Human-readable routing/export warnings corresponding to the labels; empty when none. |
| `trustBoundary` | string | Exact package-kind-specific text below. |

For `stillPhoto` and `livePhotoPackage`, `trustBoundary` is exactly:

```text
This sidecar is not signed. Verify primary photo and paired video bytes against the TAP signature embedded in the photo.
```

For `tapVideo`, it is exactly:

```text
This sidecar is not signed. Verify original video bytes against the TAP signature embedded in the video.
```

Producers MUST NOT add capture IDs, package or Photos identifiers, App Attest
key IDs, assertions, signing bindings, proof bodies, content digests, resource
hashes, backend requests or verification results. Readers ignore such extras;
no sidecar value is proof. Presentation fields and their spelling, order or
presence are not signature conditions.

The sidecar family is routed by the exact pair `schemaID` plus `version`. A
missing, malformed, unknown, superseded, or cross-family value fails closed;
field similarity and legacy aliases do not permit fallback.

## Resource descriptors

Producers write all these fields in every `resources` element:

| Field | Type | Meaning |
| --- | --- | --- |
| `role` | string | Exact role identifier below. |
| `filename` | non-empty string | Exact archive-entry path to resolve. |
| `mediaType` | non-empty string | Presentation-only media type. |

V1 roles are:

| Role | Cardinality | Declared resource | V1 media type |
| --- | --- | --- | --- |
| `primaryPhoto` | exactly one in Still/Live Photo; absent for TAP Video | Original proof-bearing HEIC/HEIF or JPEG bytes | Producer uses `public.heic` for HEIC and `public.jpeg` for JPEG. |
| `pairedLivePhotoVideo` | exactly one for `livePhotoPackage`; absent otherwise | Original paired QuickTime MOV bytes | `com.apple.quicktime-movie` |
| `primaryVideo` | exactly one for `tapVideo`; absent otherwise | Complete original proof-bearing MP4 bytes | `public.mpeg-4` |

The table defines producer output. A reader recognizes the three roles above,
ignores other roles, and selects one unique primary with at most one paired MOV
for a photo. Descriptor order and descriptive `packageKind` do not select the
artifact's signing family.

`filename` resolves the exact archive entry. A verifier MUST NOT select the
first file with a matching extension, guess a conventional name after a failed
lookup, or silently replace one declared entry with another. Producers use the
suffixes shown above. Verification uses the declared role and actual primary
container bytes; suffix and descriptive `mediaType` do not authenticate content.

## Fail-closed package resolution

After checking the transport identity and sidecar family above, a verifier
MUST reject the package when:

- the archive is malformed or violates the verifier's bounded extraction
  limits;
- the root `tapcam-export.json` entry is missing, duplicated, too large,
  non-UTF-8, non-JSON, or not an object;
- the resource list is missing or malformed, has no primary or mixed photo/video
  primary roles, duplicates a recognized role/path, or pairs a MOV with video;
- a descriptor has an empty or unsafe filename, or resolves ambiguously; or
- the primary descriptor names a missing archive entry.

Every descriptor requires a string role and a safe root-entry filename, even
when its role is ignored. Archive entries must be unique. A missing declared
MOV is returned as absent; a present MOV, including an empty file, is returned
unchanged. The [binding verifier](../bindings/capture-binding-and-proof-v1.md#local-artifact-binding-gate)
owns hash comparison and the resulting Live Photo scope.

## Bounded-input behavior

Consumers MUST bound archive bytes, entry count, individual entry sizes,
aggregate extracted bytes, and sidecar bytes before or during extraction. Exact
safety budgets are consumer-local policy; they are not signed fields, wire-format
limits, or evidence about the media.

Only the root sidecar and supported photo/MOV/MP4 candidates need to be
materialized. Paths MUST be resolved as archive entries, not written to
arbitrary filesystem locations.
