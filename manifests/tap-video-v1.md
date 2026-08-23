# TAP Video Manifest v1

Status: current v1 producer contract
Artifact: one TAP Video MP4
Example: [`../examples/manifests/tap-video-v1.json`](../examples/manifests/tap-video-v1.json)

This document defines the complete manifest written for one current TAP Video
artifact. The JSON is embedded in the MP4; it is not a `.tapnap` sidecar. MP4
placement and the timed-depth track are defined in
[`../containers/tap-video-container-v1.md`](../containers/tap-video-container-v1.md).

## Family and serialization

The three schema values are an indivisible family identity:

| JSON path | Required value |
| --- | --- |
| `schema.id` | `urn:tapnap:tapcam:video-manifest:v1` |
| `schema.version` | integer `1` |
| `schema.mediaType` | `application/vnd.tapnap.video-manifest+json;version=1` |

A consumer MUST match all three values exactly and MUST reject a missing,
unknown, superseded, or cross-family combination. Similar field names do not
permit fallback to a Still Photo or Live Photo family.

The top-level object contains exactly these contract fields:

| Field | Type | Presence | Meaning |
| --- | --- | --- | --- |
| `schema` | object | required | Exact family identity above. |
| `payload` | object | required | Signed capture and finalized-container facts. |
| `proofs` | array | required, empty | MUST be `[]`. The proof envelope belongs only in the fixed MP4 proof slot. |

The current producer serializes JSON with keys sorted recursively and without
escaping `/`. The metadata hash media type is
`application/vnd.tapnap.video-manifest.payload+json;version=1`, and its bytes
are the producer's canonical encoding of `payload` alone. `proofs` is excluded
from that metadata hash. The complete raw manifest box remains covered by the
MP4 asset hash. This is the current encoder convention; it is not a claim of
RFC 8785 conformance.

In the tables below:

- **required** means the current producer contract requires the key and a
  non-`null` value.
- **nullable** means the key is required and JSON `null` has the stated
  unavailable meaning.
- **optional** means the current Swift writer omits the key when its value is
  unavailable. Existing current-v1 decoding and the current golden vector also
  treat an explicit JSON `null` as unavailable. A producer SHOULD use the
  current omission form; a consumer MUST treat omitted and `null` equivalently
  for these specifically marked fields.

All counts are non-negative integers unless a stricter rule is stated. All
JSON numbers that represent measured values MUST be finite.

## Payload groups

Every group named here is required in `payload`.

| Field | Type | Meaning |
| --- | --- | --- |
| `id` | non-empty string | Capture identifier. It is also the `captureID` bound by the TAP Video content digest. |
| `packageID` | string | Package identity. The current producer writes a UUID string and requires it to match the queue record before signing. |
| `capturedAt` | string | UTC ISO 8601 timestamp with fractional seconds, emitted by the same TAP date formatter as the photo families. |
| `selectedCameraPlan` | object | Resolved capture-device and focal-plan facts. |
| `container` | object | Finalized MP4 facts. |
| `rgbTrack` | object | The one RGB video track. |
| `audioTrack` | object | Captured, intentionally omitted, or unavailable audio state. |
| `depthCoverage` | object | Stored timed-depth track, format, counters, and gaps, including the canonical zero-depth case. |
| `spatialRegistration` | object | Calibration coverage and the optional reproducible depth-to-RGB mapping. |
| `synchronization` | object | RGB/depth timestamp relationship. |
| `stop` | object | Why recording stopped and the finalized duration. |
| `software` | object | Producer build and schema-writer facts. |

### `selectedCameraPlan`

| Field | Type | Presence | Unit / values |
| --- | --- | --- | --- |
| `deviceUniqueID` | string | optional | Opaque capture-device identifier. Synthetic examples omit it. |
| `deviceType` | string | optional | Producer platform device-type identifier. |
| `localizedName` | string | optional | Producer platform display name. |
| `position` | string | required | Current producer values: `front`, `back`, `unspecified`, or `unknown`. |
| `requestedFocalLengthLabel` | string | optional | Requested UI focal label, such as `24mm`; it is a label, not a numeric unit field. |
| `resolvedFocalLengthLabel` | string | optional | Resolved focal label. |
| `resolvedZoomFactor` | number | optional | Dimensionless video zoom factor. |
| `depthCapable` | boolean | required | Whether the resolved capture configuration supported depth delivery. This is capability, not proof that depth samples were stored. |

### `container`

| Field | Type | Presence | Unit / required value |
| --- | --- | --- | --- |
| `fileType` | string | required | `mp4` |
| `mediaType` | string | required | `video/mp4` |
| `durationSeconds` | number | required | seconds; `>= 0`; finalized presentation duration |
| `timeScale` | integer | required | ticks per second; `> 0` |
| `trackCount` | integer | required | actual total MP4 track count |

The finalized file and these facts MUST agree. The current composition is one
RGB video track, zero or one audio track, and zero or one TAP timed-metadata
depth track.

### `rgbTrack`

| Field | Type | Presence | Unit / meaning |
| --- | --- | --- | --- |
| `trackID` | integer | required | Actual non-conflicting MP4 track ID. |
| `codec` | non-empty string | required | Actual video sample-entry FourCC, for example `avc1`; readers MUST use the recorded fact rather than assume H.264. |
| `width` | integer | required | coded pixels; `> 0` |
| `height` | integer | required | coded pixels; `> 0` |
| `durationSeconds` | number | required | seconds; `>= 0` |
| `timeScale` | integer | required | ticks per second; `> 0` |
| `nominalFrameRate` | number | optional | frames per second |
| `frameCount` | integer | optional | captured RGB frame count |
| `transform` | string | optional | Recorded display transform. Current forms include `rotation:N`, `rotation:N;mirrored`, and the current v1 vector's `rotation:N;not-mirrored`, where `N` is `0`, `90`, `180`, or `270` degrees clockwise. |

### `audioTrack`

All seven keys are serialized. The six track-fact values are nullable.

| Field | Type | Presence | Unit / values |
| --- | --- | --- | --- |
| `status` | string | required | `captured`, `notCaptured`, or `unavailable` |
| `trackID` | integer or `null` | nullable | Actual MP4 track ID. |
| `codec` | string or `null` | nullable | Actual sample-entry codec; current captured audio uses AAC. |
| `durationSeconds` | number or `null` | nullable | seconds |
| `timeScale` | integer or `null` | nullable | ticks per second |
| `sampleRate` | number or `null` | nullable | samples per second (Hz) |
| `channelCount` | integer or `null` | nullable | audio channels |

For `captured`, `trackID`, `codec`, `durationSeconds`, and `timeScale` MUST be
non-`null`, the duration MUST be `>= 0`, and the timescale MUST be `> 0`.
`sampleRate` and `channelCount` may remain `null` if the finalized file does not
expose them. For `notCaptured` or `unavailable`, no audio track may exist and
all six fact values MUST be `null`; a producer MUST NOT invent a silent track.

### `depthCoverage`

All top-level keys in this group are serialized. Track facts and `format` are
nullable so the same family represents a canonical zero-depth capture.

| Field | Type | Presence | Unit / meaning |
| --- | --- | --- | --- |
| `trackID` | integer or `null` | nullable | Timed-metadata MP4 track ID. |
| `trackCodec` | string or `null` | nullable | Current stored-depth value: `mebx`. |
| `trackDurationSeconds` | number or `null` | nullable | seconds |
| `trackTimeScale` | integer or `null` | nullable | ticks per second |
| `sampleCount` | integer | required | Successfully stored KLV depth samples. |
| `deliveredSampleCount` | integer | required | Depth samples delivered to the recorder; MUST be `>= sampleCount`. |
| `outputDropCount` | integer | required | Capture-output drops. |
| `encodingDropCount` | integer | required | Depth packing or compression failures. |
| `metadataDropCount` | integer | required | Timed-metadata append/backpressure drops. |
| `gapCount` | integer | required | MUST equal `gaps.length`. |
| `gaps` | array of `DepthGap` | required | Truthful missing-depth intervals; at most 1,024 entries. |
| `format` | object or `null` | nullable | Invariant packed-frame layout. |

If `sampleCount > 0`, `trackID`, `trackCodec`, `trackDurationSeconds`,
`trackTimeScale`, and `format` MUST be non-`null`; duration MUST be `>= 0`; and
timescale MUST be `> 0`. The metadata track and manifest facts MUST match.

If `sampleCount == 0`, the canonical values are `trackID: null`,
`trackCodec: null`, `trackDurationSeconds: null`, `trackTimeScale: null`, and
`format: null`, and no TAP timed-depth track exists. The capture remains a TAP
Video. `gaps` may truthfully describe the affected interval, usually as
`silentCadence`; no fake sample may be inserted.

#### `depthCoverage.format`

| Field | Type | Presence | Unit / values |
| --- | --- | --- | --- |
| `kind` | string | required | `depth` or `disparity` |
| `pixelFormat` | string | required | `hdep`, `fdep`, `hdis`, or `fdis` |
| `width` | integer | required | samples (pixels) per row; `> 0` |
| `height` | integer | required | rows; `> 0` |
| `packedRowStride` | integer | required | bytes; MUST equal `width * bytesPerSample` |
| `sourceRowStride` | integer | optional | capture-buffer bytes per row; if present, MUST be at least `packedRowStride`; it is diagnostic padding, not stored pixel content |
| `bytesPerSample` | integer | required | `2` for `hdep`/`hdis`; `4` for `fdep`/`fdis` |
| `byteOrder` | string | required | `little-endian` |
| `uncompressedFrameByteCount` | integer | required | bytes; MUST equal `packedRowStride * height` and MUST be at most 32 MiB |
| `compressionPolicy` | string | required | Current writer: `per-frame:zstd1|raw`; current v1 readers also recognize `per-frame:lzfse|raw` and `per-frame:raw`. Every KLV `COMP` value MUST be allowed by this policy. |

The valid `(kind, pixelFormat, bytesPerSample)` combinations are exactly
`(depth,hdep,2)`, `(depth,fdep,4)`, `(disparity,hdis,2)`, and
`(disparity,fdis,4)`. Rows contain only logical samples; source-buffer padding
is not copied into `DPTH`.

#### `DepthGap`

| Field | Type | Presence | Unit / values |
| --- | --- | --- | --- |
| `reason` | string | required | `outputDrop`, `encodingFailure`, `metadataBackpressure`, `silentCadence`, or `boundedAggregation` |
| `startPTS` | `MediaTime` | required | inclusive capture-relative start |
| `endPTS` | `MediaTime` | required | capture-relative end; MUST be at or after start |
| `nearestStartRGBFrame` | integer | optional | nearest RGB frame index at the start |
| `nearestEndRGBFrame` | integer | optional | nearest RGB frame index at the end |

`boundedAggregation` is the conservative range used after the 1,024-entry gap
table reaches its limit; a consumer clears depth for the whole range. A
`MediaTime` object has required integer `value` (ticks) and integer `timescale`
(ticks per second, `> 0`); seconds are `value / timescale`.

### `spatialRegistration`

| Field | Type | Presence | Unit / meaning |
| --- | --- | --- | --- |
| `status` | string | required | Current valid artifact states: `registered` or `unavailable`. The schema enum contains `approximate`, but current validation rejects it; it MUST NOT enable an overlay. |
| `mapping` | string | required | Registration family identifier when registered, otherwise `unavailable` or a producer reason prefixed by `avdepthdata-registration-prerequisites-unavailable:`. |
| `rgbReferenceDimensions` | `Dimensions` | optional | aligned RGB coded pixels |
| `depthReferenceDimensions` | `Dimensions` | optional | depth pixels |
| `rgbCleanAperture` | `Rect` | optional | encoded RGB pixel coordinates, origin at top left |
| `recordedTransform` | string | optional | recorded RGB connection/display transform |
| `calibrationTable` | array | required | At most 16 `CameraCalibration` entries. |
| `calibrationCoverage` | object | required | How stored depth samples reference the table. |
| `descriptor` | object | optional | Complete reproducible mapping; required only for `registered`. |

For `registered`, `mapping` MUST equal
`urn:tapnap:tapcam:video-depth-registration:avdepthdata-yuv-warp:v1` and
`descriptor` MUST be present and valid. For `unavailable`, `descriptor` MUST be
absent or `null`. `approximate` is not a valid substitute for `registered`.

`Dimensions` has required numeric `width` and `height`, in pixels. `Rect` has
required numeric `x`, `y`, `width`, and `height`, in top-left-origin pixels.
`Point` has required numeric `x` and `y`, in pixels in the associated reference
image.

#### `calibrationCoverage`

| Field | Type | Presence | Meaning |
| --- | --- | --- | --- |
| `indexedSampleCount` | integer | required | Samples carrying a valid `CALI` index. |
| `missingCalibrationSampleCount` | integer | required | Samples without calibration because none was supplied. |
| `overflowUnindexedSampleCount` | integer | required | Samples left unindexed after the 16-entry table filled. |
| `tableOverflowed` | boolean | required | Whether additional distinct calibration could not be indexed. |

The three counts MUST sum to `depthCoverage.sampleCount`. A positive indexed
count requires a non-empty table. A positive overflow count requires
`tableOverflowed: true`; when `tableOverflowed` is true, the table has 16
entries.

#### `CameraCalibration`

| Field | Type | Presence | Unit / representation |
| --- | --- | --- | --- |
| `intrinsicMatrix` | array of 9 numbers | required | AVFoundation 3x3 intrinsic matrix flattened column by column; image terms use the associated reference-pixel coordinate system. |
| `intrinsicMatrixReferenceDimensions` | `Dimensions` | required | reference pixels |
| `extrinsicMatrix` | array of 12 numbers | required | AVFoundation 3x4 extrinsic matrix flattened column by column; values are copied without unit conversion. |
| `pixelSizeMillimeters` | number | required | millimetres per sensor pixel |
| `lensDistortionCenter` | `Point` | required | pixels in the intrinsic reference image |
| `lensDistortionLookupTable` | base64 string | optional | Raw AVFoundation lookup-table bytes encoded by JSON as base64. |
| `inverseLensDistortionLookupTable` | base64 string | optional | Raw inverse lookup-table bytes encoded by JSON as base64. |

The manifest does not reinterpret or change the coordinate convention of the
AVFoundation calibration values.

#### Registration descriptor

All descriptor fields are required:

| Field | Type | Required value / meaning |
| --- | --- | --- |
| `schema` | string | `urn:tapnap:tapcam:video-depth-registration:avdepthdata-yuv-warp:v1` |
| `version` | integer | `1` |
| `model` | string | `avdepthdata-warped-to-synchronized-rgb-pixel-centers` |
| `alignedRGBCodedDimensions` | `Dimensions` | RGB dimensions after quarter-turn alignment, pixels |
| `encodedRGBCodedDimensions` | `Dimensions` | encoded RGB dimensions, pixels |
| `depthDimensions` | `Dimensions` | packed depth dimensions, pixels |
| `depthToAlignedRGBPixelCenterAffine` | array of 6 numbers | `[a,b,tx,c,d,ty]`, mapping a depth pixel centre `(x,y)` to aligned RGB pixel centre `(a*x+b*y+tx, c*x+d*y+ty)`; scale terms are dimensionless and translations are pixels |
| `connectionTransform` | string | `rotation:N;mirrored` or `rotation:N;not-mirrored`, with `N` one of `0`, `90`, `180`, `270` degrees |
| `isEncodedHorizontallyMirrored` | boolean | Whether the encoded RGB is horizontally mirrored |
| `rgbCleanAperture` | `Rect` | presented RGB aperture in encoded, top-left-origin pixels |
| `videoStabilizationMode` | string | `off` |

The model means `AVDepthData` has already been lens-warped into the synchronized
pre-connection RGB coordinate system. Apply the affine to depth pixel centres,
then the connection rotation and optional horizontal mirror in encoded space,
then the clean aperture. Only a fully validated `registered` descriptor enables
registered 2D playback.

### `synchronization`

| Field | Type | Presence | Unit / meaning |
| --- | --- | --- | --- |
| `timing` | string | required | Current writer: `capture-output-presentation-timestamps`. |
| `rgbToDepthMapping` | string | required | Current stored-depth writer: `independent-timed-metadata`; canonical zero-depth value: `no-depth-samples`. |
| `maxObservedDeltaSeconds` | number | optional | seconds; maximum observed synchronized RGB/depth delivery delta |
| `maxObservedDepthIntervalSeconds` | number | optional | seconds; maximum interval between stored depth samples |
| `nominalDepthIntervalSeconds` | number | optional | seconds; expected source depth cadence |

The descriptive strings do not replace sample timestamps. Stored KLV PTS values
MUST be strictly increasing, agree with their timed-metadata group timestamps,
and remain within the signed track duration. Discontinuities must be covered by
the signed gap table.

### `stop`

| Field | Type | Presence | Unit / values |
| --- | --- | --- | --- |
| `reason` | string | required | `userStop`, `durationLimit`, `thermalPressure`, `systemPressure`, `appLifecycle`, `storageFailure`, or `captureFailure` |
| `recordedDurationSeconds` | number | required | seconds; finalized recorded duration, `>= 0` |

The current UI's 180-second default stop is runtime policy, not a v1 decoder
limit.

### `software`

| Field | Type | Presence | Meaning |
| --- | --- | --- | --- |
| `appIdentifier` | string | required | Producer application identifier. |
| `appVersion` | string | required | Producer short version. |
| `buildNumber` | string | required | Producer build number. |
| `schemaWriter` | string | required | Current producer value: `TAPCamDemo.TAPVideoManifestEncoder`. |

## Consumer obligations and extraction note

A consumer MUST first route the exact family and require `proofs: []`. It then
authenticates the MP4 and canonical payload through the TAP Video content
binding before treating any payload claim as authenticated. A valid manifest
does not by itself prove the physical scene, event, person, time, non-AI origin,
or depth correctness.

The current TypeScript Verifier strictly checks the complete schema identity,
empty `proofs`, `payload.id`, `payload.packageID`, `payload.capturedAt`, and
`depthCoverage.sampleCount`, but presently retains many other payload groups as
untyped data rather than validating the complete producer field contract. That
looser implementation is an extraction divergence, not permission for a v1
producer to omit required groups or change their units, nullability, or
semantics. See [`../KNOWN_DIVERGENCES.md`](../KNOWN_DIVERGENCES.md).
