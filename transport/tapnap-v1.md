# `.tapnap` Verification Transport v1

Status: current v1 transport contract for Still Photo and Live Photo
Routing sidecar family: `urn:tapnap:tapcam:verification-export:v1`

`.tapnap` is a byte-preserving transport wrapper. It carries original media to
a verifier; it is not an artifact-manifest family, a signature format, or
authenticity evidence. The authoritative Still Photo or Live Photo manifest and
proof remain embedded in the primary HEIC/JPEG resource.

Current product support is deliberately bounded:

| Artifact | `.tapnap` status |
| --- | --- |
| Still Photo v1 | current |
| Live Photo v1 | current |
| TAP Video v1 | future / Coming Soon |

A current TAP Video is one signed MP4 and MUST NOT be wrapped in `.tapnap` or
routed with `tapcam-export.json`. This document assigns no Video package kind or
Video resource role.

## Transport identity

| Identifier | Current value |
| --- | --- |
| Preferred filename | `TAPNAP-Capture.tapnap` |
| Filename extension | `.tapnap` |
| UTI | `net.tapnap.capture-package` |
| MIME type | `application/vnd.tapnap.capture-package+zip` |
| Container | ZIP-compatible archive |

A verifier recognizes a capture package only from the `.tapnap` extension or
the exact TAPNAP MIME type. Generic `.zip`, `application/zip`, ZIP magic, a
legacy marker, or archive contents MUST NOT silently reclassify an input as a
current TAPNAP package. Raw HEIC/HEIF/JPG/JPEG and TAP Video MP4 inputs remain
their own input routes.

The current producer writes media bytes unchanged and writes archive entries
without compression. A verifier MUST NOT transcode or normalize declared media
before embedded-proof and resource-hash verification.

## Current archive layouts

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

The sidecar is exactly the root entry `tapcam-export.json`. Current producers
write the declared media resources as root entries with the names above. A
Live Photo whose original paired video is unavailable is not a successful Live
Photo package; a separately shared primary-photo-only fallback remains ordinary
media and must report that full Live Photo verification is incomplete.

## `tapcam-export.json`

The sidecar is unsigned routing metadata. Its only authority is to map a small
set of resource roles to exact archive entries. A verifier derives the actual
manifest family and verification scope from the proof-bearing primary photo,
never from `packageKind`, `mediaType`, warnings, filenames, or archive shape.

Every current producer sidecar contains these fields:

| Field | Type | Presence | Required value / meaning |
| --- | --- | --- | --- |
| `schemaID` | string | required | `urn:tapnap:tapcam:verification-export:v1` |
| `version` | integer | required | `1` |
| `packageKind` | string | required | `stillPhoto` or `livePhotoPackage` |
| `resources` | array | required | Exact role-to-entry mappings defined below. |
| `warningLabels` | array of strings | required | Labels for presentation-adjustment resources detected by the producer; empty when none. |
| `warnings` | array of strings | required | Human-readable routing/export warnings corresponding to the labels; empty when none. These strings are not signed claims. |
| `trustBoundary` | string | required | `This sidecar is not signed. Verify primary photo and paired video bytes against the TAP signature embedded in the photo.` |

A conforming v1 sidecar contains only these routing/presentation fields. It
MUST NOT contain capture IDs, package or Photos identifiers, App Attest key IDs,
assertion objects, signing bindings, proof bodies, content digests, resource
hashes, backend requests, or server verification results. Moving such values
into an unsigned sidecar would not authenticate them.

The sidecar family is routed by the exact pair `schemaID` plus `version`. A
missing, malformed, unknown, superseded, or cross-family value fails closed;
field similarity and legacy aliases do not permit fallback.

## Resource descriptors

Every `resources` element has exactly these current fields:

| Field | Type | Presence | Meaning |
| --- | --- | --- | --- |
| `role` | string | required | Exact role identifier below. |
| `filename` | non-empty string | required | Exact archive-entry path to resolve. |
| `mediaType` | non-empty string | required | Producer-declared media type for routing and presentation only. |

Current roles are:

| Role | Cardinality | Declared resource | Current media type |
| --- | --- | --- | --- |
| `primaryPhoto` | exactly one in every package | Original proof-bearing HEIC/HEIF or JPEG bytes | Producer uses `public.heic` for HEIC and `public.jpeg` for JPEG. |
| `pairedLivePhotoVideo` | zero for `stillPhoto`; exactly one for `livePhotoPackage` | Original paired QuickTime MOV bytes | `com.apple.quicktime-movie` |

`tapDepthManifestPayload` is a signed-resource role inside the embedded Live
Photo content digest. It is not a `.tapnap` archive resource and MUST NOT appear
as a sidecar resource descriptor.

For a conforming `stillPhoto` sidecar, `resources` contains exactly one
`primaryPhoto` descriptor and no paired-video descriptor. For a conforming
`livePhotoPackage` sidecar, it contains exactly one `primaryPhoto` followed by
exactly one `pairedLivePhotoVideo`. The `packageKind` and resource set MUST
agree, but neither is allowed to override the embedded manifest family.

`filename` resolves the exact archive entry. A verifier MUST NOT select the
first file with a matching extension, guess a conventional name after a failed
lookup, or silently replace one declared entry with another. The primary entry
must have a supported photo suffix (`.heic`, `.heif`, `.jpg`, or `.jpeg`), and
the paired resource must have `.mov`; suffix and declared `mediaType` are routing
checks, not authenticity checks.

## Fail-closed package resolution

A verifier MUST reject the package as current-v1 input when any of the following
is true:

- the input was reached only through generic ZIP detection rather than the
  `.tapnap` extension or exact MIME type;
- the archive is malformed or violates the verifier's bounded extraction
  limits;
- the root `tapcam-export.json` entry is missing, duplicated, too large,
  non-UTF-8, non-JSON, or not an object;
- `schemaID` or `version` is missing or not the exact current v1 value;
- a complete producer field is missing or has the wrong JSON type;
- `packageKind` is not `stillPhoto` or `livePhotoPackage`;
- `resources` contains zero or multiple `primaryPhoto` descriptors;
- a Still Photo declares a paired-video role, or a Live Photo does not declare
  exactly one paired-video role;
- a role is unknown, duplicated, or inconsistent with the package kind;
- a descriptor has an empty filename/media type, names a missing entry, names
  the wrong supported media suffix, or resolves ambiguously; or
- the sidecar attempts to supply authenticity, signing, hash, or server-result
  fields.

Package-routing failure and artifact-verification failure are separate. If the
sidecar resolves a structurally valid package, the verifier still MUST parse the
primary photo's embedded manifest and proof, recompute the signed byte ranges,
and, for a Live Photo, hash the complete declared MOV against
`signedResources.pairedLivePhotoVideo`. A sidecar match alone never produces a
valid verification result.

## Bounded-input behavior

Consumers MUST bound archive bytes, entry count, individual entry sizes,
aggregate extracted bytes, and sidecar bytes before or during extraction. The
current browser verifier uses implementation safety budgets of 512 MiB archive
bytes, 16 archive entries, 384 MiB per media resource, 512 MiB aggregate
extracted bytes, and 256 KiB for `tapcam-export.json`. These are consumer safety
limits, not new signed fields and not evidence about the media.

Only the root sidecar and supported photo/MOV candidates need to be
materialized. Paths MUST be resolved as archive entries, not written to
arbitrary filesystem locations.

Current verifier coverage gaps are tracked only in
[Known Extraction Divergences](../KNOWN_DIVERGENCES.md).
