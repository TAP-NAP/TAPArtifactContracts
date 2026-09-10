# `.tapnap` Verification Transport v1

Status: current pre-release v1 transport contract for Still Photo, Live Photo, and TAP Video
Routing sidecar family: `urn:tapnap:tapcam:verification-export:v1`

`.tapnap` is a byte-preserving transport wrapper. It carries original media to
a verifier; it is not an artifact-manifest family, a signature format, or
authenticity evidence. The authoritative manifest and proof remain embedded in
the original media: the primary HEIC/JPEG for Still/Live Photo, or the MP4 for
TAP Video.

V1 support is deliberately bounded:

| Artifact | `.tapnap` status |
| --- | --- |
| Still Photo v1 | supported |
| Live Photo v1 | supported |
| TAP Video v1 | supported |

A TAP Video package carries one complete original signed MP4. Its embedded
manifest, proof slot, RGB/audio tracks, and any depth or telemetry remain inside
that MP4. Producers MUST NOT transcode the media or extract a second manifest,
proof, or depth file for this transport. Raw MP4 sharing remains supported.

This video layout is part of the current pre-release v1 definition under the
[pre-release policy](../VERSIONING.md#current-pre-release-policy). It extends the
earlier photo-only development definition; this does not imply that earlier
producer or verifier revisions supported video packages.

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

Still Photo:

```text
TAPNAP-Capture.tapnap
├── primary-photo.heic   or primary-photo.jpg
└── tapcam-export.json
```

Live Photo:

```text
TAPNAP-Capture.tapnap
├── primary-photo.heic   or primary-photo.jpg
├── paired-video.mov
└── tapcam-export.json
```

TAP Video:

```text
TAPNAP-Capture.tapnap
├── original-video.mp4
└── tapcam-export.json
```

The sidecar is exactly the root entry `tapcam-export.json`. V1 producers write
the declared media resources as root entries with the names above. A
Live Photo whose original paired video is unavailable is not a successful Live
Photo package; a separately shared primary-photo-only fallback remains ordinary
media and must report that full Live Photo verification is incomplete.

## `tapcam-export.json`

The sidecar is unsigned routing metadata. Its only authority is to map a small
set of resource roles to exact archive entries. A verifier derives the actual
manifest family and verification scope from the proof-bearing original media,
never from `packageKind`, `mediaType`, warnings, filenames, or archive shape.
`packageKind` selects the bounded media parser; an incompatible or invalid
embedded artifact fails verification rather than being reclassified.

Every v1 producer sidecar contains these fields:

| Field | Type | Presence | Required value / meaning |
| --- | --- | --- | --- |
| `schemaID` | string | required | `urn:tapnap:tapcam:verification-export:v1` |
| `version` | integer | required | `1` |
| `packageKind` | string | required | `stillPhoto`, `livePhotoPackage`, or `tapVideo` |
| `resources` | array | required | Exact role-to-entry mappings defined below. |
| `warningLabels` | array of strings | required | Labels for presentation-adjustment resources detected by the producer; empty when none. |
| `warnings` | array of strings | required | Human-readable routing/export warnings corresponding to the labels; empty when none. These strings are not signed claims. |
| `trustBoundary` | string | required | The exact package-kind-specific text below. |

For `stillPhoto` and `livePhotoPackage`, `trustBoundary` is exactly:

```text
This sidecar is not signed. Verify primary photo and paired video bytes against the TAP signature embedded in the photo.
```

For `tapVideo`, it is exactly:

```text
This sidecar is not signed. Verify original video bytes against the TAP signature embedded in the video.
```

A conforming v1 sidecar contains only these routing/presentation fields. It
MUST NOT contain capture IDs, package or Photos identifiers, App Attest key IDs,
assertion objects, signing bindings, proof bodies, content digests, resource
hashes, backend requests, or server verification results. Moving such values
into an unsigned sidecar would not authenticate them.

The sidecar family is routed by the exact pair `schemaID` plus `version`. A
missing, malformed, unknown, superseded, or cross-family value fails closed;
field similarity and legacy aliases do not permit fallback.

## Resource descriptors

Every `resources` element has exactly these v1 fields:

| Field | Type | Presence | Meaning |
| --- | --- | --- | --- |
| `role` | string | required | Exact role identifier below. |
| `filename` | non-empty string | required | Exact archive-entry path to resolve. |
| `mediaType` | non-empty string | required | Producer-declared media type for routing and presentation only. |

V1 roles are:

| Role | Cardinality | Declared resource | V1 media type |
| --- | --- | --- | --- |
| `primaryPhoto` | exactly one in Still/Live Photo; absent for TAP Video | Original proof-bearing HEIC/HEIF or JPEG bytes | Producer uses `public.heic` for HEIC and `public.jpeg` for JPEG. |
| `pairedLivePhotoVideo` | exactly one for `livePhotoPackage`; absent otherwise | Original paired QuickTime MOV bytes | `com.apple.quicktime-movie` |
| `primaryVideo` | exactly one for `tapVideo`; absent otherwise | Complete original proof-bearing MP4 bytes | `public.mpeg-4` |

`tapDepthManifestPayload` is a signed-resource role inside the embedded Live
Photo content digest. It is not a `.tapnap` archive resource and MUST NOT appear
as a sidecar resource descriptor.

For a conforming `stillPhoto` sidecar, `resources` contains exactly one
`primaryPhoto` descriptor and no paired-video descriptor. For a conforming
`livePhotoPackage` sidecar, it contains exactly one `primaryPhoto` followed by
exactly one `pairedLivePhotoVideo`. For `tapVideo`, it contains exactly one
`primaryVideo` and no photo or paired-video descriptors. The `packageKind` and
resource set MUST agree, but neither may override the embedded manifest family.

`filename` resolves the exact archive entry. A verifier MUST NOT select the
first file with a matching extension, guess a conventional name after a failed
lookup, or silently replace one declared entry with another. The primary entry
must have a supported photo suffix (`.heic`, `.heif`, `.jpg`, or `.jpeg`), and
the paired resource must have `.mov`. A `primaryVideo` resource must have `.mp4`.
Suffix and declared `mediaType` are routing checks, not authenticity checks.

## Fail-closed package resolution

A verifier MUST reject the package as v1 input when any of the following
is true:

- the input was reached only through generic ZIP detection rather than the
  `.tapnap` extension or exact MIME type;
- the archive is malformed or violates the verifier's bounded extraction
  limits;
- the root `tapcam-export.json` entry is missing, duplicated, too large,
  non-UTF-8, non-JSON, or not an object;
- `schemaID` or `version` is missing or not the exact v1 value;
- a complete producer field is missing or has the wrong JSON type;
- `packageKind` is not `stillPhoto`, `livePhotoPackage`, or `tapVideo`;
- the exact resource roles, cardinality, or ordering do not match the package
  kind, including mixed photo/video primary roles;
- a role is unknown, duplicated, or inconsistent with the package kind;
- a descriptor has an empty filename/media type, names a missing entry, names
  the wrong supported media suffix, or resolves ambiguously; or
- the sidecar attempts to supply authenticity, signing, hash, or server-result
  fields.

Package-routing failure and artifact-verification failure are separate. If the
sidecar resolves a structurally valid package, the verifier still MUST parse the
original media's embedded manifest and proof and recompute the signed byte
ranges. Live Photo also hashes the complete declared MOV against
`signedResources.pairedLivePhotoVideo`. TAP Video follows exactly the raw MP4
verification path, including its applicable container/KLV semantic gates.
A sidecar match alone never produces a valid verification result. Zero depth is
not a package-routing failure and must not prevent a valid signed video from
being transported or verified.

## Bounded-input behavior

Consumers MUST bound archive bytes, entry count, individual entry sizes,
aggregate extracted bytes, and sidecar bytes before or during extraction. Exact
safety budgets are consumer-local policy; they are not signed fields, wire-format
limits, or evidence about the media.

Only the root sidecar and supported photo/MOV/MP4 candidates need to be
materialized. Paths MUST be resolved as archive entries, not written to
arbitrary filesystem locations.
