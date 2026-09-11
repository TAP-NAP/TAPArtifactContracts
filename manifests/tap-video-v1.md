# TAP Video Manifest v1

Status: v1 data representation
Artifact: one TAP Video MP4
Example: [`../examples/manifests/tap-video-v1.json`](../examples/manifests/tap-video-v1.json)

This document defines the producer's manifest fields, embedded in one MP4.
[TAP Video Container](../containers/tap-video-container-v1.md) defines byte
layout and decoding; [Capture Binding and Proof](../bindings/capture-binding-and-proof-v1.md)
defines hashing and verification.

## Family and serialization

Producers write these schema values:

| JSON path | Required value |
| --- | --- |
| `schema.id` | `urn:tapnap:tapcam:video-manifest:v1` |
| `schema.version` | integer `1` |
| `schema.mediaType` | `application/vnd.tapnap.video-manifest+json;version=1` |

Producers write these top-level fields:

| Field | Type | Presence | Meaning |
| --- | --- | --- | --- |
| `schema` | object | required | Exact family identity above. |
| `payload` | object | required | Signed capture and finalized-container facts. |
| `proofs` | array | required, empty | Producer writes `[]`. |

Producers encode the manifest and payload using
[TAP capture canonical JSON](../bindings/capture-binding-and-proof-v1.md#tap-capture-canonical-json).
In the tables below:

- **required** means the key and a non-`null` value are required.
- **nullable** means the key is required and JSON `null` has the stated
  unavailable meaning.
- **optional** means a v1 producer omits the key when its value is unavailable.
  A producer SHOULD use that omission form; a consumer MUST treat omission and
  explicit JSON `null` equivalently for these specifically marked fields.

## Payload groups

Every group named here is required in `payload`.

| Field | Type | Meaning |
| --- | --- | --- |
| `id` | non-empty string | Capture identifier. It is also the `captureID` bound by the TAP Video content digest. |
| `packageID` | string | Opaque package identity recorded by the producer. |
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
| `position` | string | required | `front`, `back`, `unspecified`, or `unknown`. |
| `requestedFocalLengthLabel` | string | optional | Requested UI focal label, such as `24mm`; it is a label, not a numeric unit field. |
| `resolvedFocalLengthLabel` | string | optional | Resolved focal label. |
| `resolvedZoomFactor` | number | optional | Dimensionless video zoom factor. |
| `depthCapable` | boolean | required | Whether the resolved capture configuration supported depth delivery. This is capability, not proof that depth samples were stored. |

### `container`

| Field | Type | Presence | Unit / required value |
| --- | --- | --- | --- |
| `fileType` | string | required | `mp4` |
| `mediaType` | string | required | `video/mp4` |
| `durationSeconds` | number | required | seconds; finalized presentation duration, `>= 0` |
| `timeScale` | integer | required | ticks per second, `> 0` |
| `trackCount` | integer | required | observed total MP4 track count |

These facts come from the finalized MP4's postflight inspection.

### `rgbTrack`

| Field | Type | Presence | Unit / meaning |
| --- | --- | --- | --- |
| `trackID` | integer | required | Observed MP4 track ID. |
| `codec` | non-empty string | required | Observed video sample-entry FourCC, for example `avc1`. |
| `width` | integer | required | coded pixels, `> 0` |
| `height` | integer | required | coded pixels, `> 0` |
| `durationSeconds` | number | required | seconds, `>= 0` |
| `timeScale` | integer | required | ticks per second, `> 0` |
| `nominalFrameRate` | number | optional | frames per second |
| `frameCount` | integer | optional | captured RGB frame count |
| `transform` | string | optional | Recorded RGB display transform. V1 forms are `rotation:N` and `rotation:N;mirrored`, where `N` is `0`, `90`, `180`, or `270` degrees clockwise. |

Display-coordinate mappings are defined in the [decoder section](../containers/tap-video-container-v1.md#display-coordinates).

### `audioTrack`

All seven keys are serialized. The six track-fact values are nullable.

| Field | Type | Presence | Unit / values |
| --- | --- | --- | --- |
| `status` | string | required | `captured`, `notCaptured`, or `unavailable` |
| `trackID` | integer or `null` | nullable | Actual MP4 track ID. |
| `codec` | string or `null` | nullable | Actual sample-entry codec. |
| `durationSeconds` | number or `null` | nullable | seconds |
| `timeScale` | integer or `null` | nullable | ticks per second |
| `sampleRate` | number or `null` | nullable | samples per second (Hz) |
| `channelCount` | integer or `null` | nullable | audio channels |

For `captured`, the producer records `trackID`, `codec`, `durationSeconds`, and
`timeScale` from its audio track.
`sampleRate` and `channelCount` may remain `null` if the finalized file does not
expose them. For `notCaptured` or `unavailable`, the producer records null track
facts.

### `depthCoverage`

All top-level keys in this group are serialized. Track facts and `format` are
nullable so the same family represents a canonical zero-depth capture.

| Field | Type | Presence | Unit / meaning |
| --- | --- | --- | --- |
| `trackID` | integer or `null` | nullable | Timed-metadata MP4 track ID. |
| `trackCodec` | string or `null` | nullable | Exact stored-depth value `mebx` when samples are present. |
| `trackDurationSeconds` | number or `null` | nullable | seconds |
| `trackTimeScale` | integer or `null` | nullable | ticks per second |
| `sampleCount` | integer | required | Successfully stored KLV depth samples. |
| `deliveredSampleCount` | integer | required | Depth samples delivered to the recorder. |
| `outputDropCount` | integer | required | Capture-output drops. |
| `encodingDropCount` | integer | required | Depth packing or compression failures. |
| `metadataDropCount` | integer | required | Timed-metadata append/backpressure drops. |
| `gapCount` | integer | required | Producer's recorded gap count. |
| `gaps` | array of `DepthGap` | required | Truthful missing-depth intervals; at most 1,024 entries. |
| `format` | object or `null` | nullable | Invariant packed-frame layout. |


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
| `compressionPolicy` | string | required | `per-frame:zstd1|raw`, `per-frame:lzfse|raw`, or `per-frame:raw`. Every KLV `COMP` value MUST be allowed by this policy. |

Packed frame layout is defined with the [codec rules](../containers/tap-video-container-v1.md#packed-frame-and-codec-rules).

#### `DepthGap`

| Field | Type | Presence | Unit / values |
| --- | --- | --- | --- |
| `reason` | string | required | `outputDrop`, `encodingFailure`, `metadataBackpressure`, `silentCadence`, or `boundedAggregation` |
| `startPTS` | `MediaTime` | required | inclusive capture-relative start |
| `endPTS` | `MediaTime` | required | Recorded capture-relative end |
| `nearestStartRGBFrame` | integer | optional | nearest RGB frame index at the start |
| `nearestEndRGBFrame` | integer | optional | nearest RGB frame index at the end |

`boundedAggregation` records a conservative missing-depth interval after the
1,024-entry table reaches its limit. A `MediaTime` object has required integer `value` (ticks) and integer `timescale`
(ticks per second, `> 0`); seconds are `value / timescale`.

### `spatialRegistration`

| Field | Type | Presence | Unit / meaning |
| --- | --- | --- | --- |
| `status` | string | required | `registered` or `unavailable`. |
| `mapping` | string | required | Registration family identifier when registered, otherwise `unavailable` or a producer reason prefixed by `avdepthdata-registration-prerequisites-unavailable:`. |
| `rgbReferenceDimensions` | `Dimensions` | optional | aligned RGB coded pixels |
| `depthReferenceDimensions` | `Dimensions` | optional | depth pixels |
| `rgbCleanAperture` | `Rect` | optional | encoded RGB pixel coordinates, origin at top left |
| `recordedTransform` | string | optional | recorded RGB connection/display transform |
| `calibrationTable` | array | required | At most 16 `CameraCalibration` entries. |
| `calibrationCoverage` | object | required | How stored depth samples reference the table. |
| `descriptor` | object | optional | Complete reproducible mapping; required only for `registered`. |

The producer omits the registration descriptor when unavailable.

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

The optional [inline-calibration `CALD` record](../containers/tap-video-container-v1.md#inline-calibration-extension-cald)
preserves a frame's calibration when the table is full. Such a frame still
counts as `overflowUnindexedSampleCount`; it creates no `CALI` index and changes
none of these counters.

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

See [display coordinates](../containers/tap-video-container-v1.md#display-coordinates) for the transform order.

### `synchronization`

| Field | Type | Presence | Unit / meaning |
| --- | --- | --- | --- |
| `timing` | string | required | Exact v1 value `capture-output-presentation-timestamps`. |
| `rgbToDepthMapping` | string | required | `independent-timed-metadata` when depth samples are stored; canonical zero-depth value `no-depth-samples`. |
| `maxObservedDeltaSeconds` | number | optional | seconds; maximum observed synchronized RGB/depth delivery delta |
| `maxObservedDepthIntervalSeconds` | number | optional | seconds; maximum interval between stored depth samples |
| `nominalDepthIntervalSeconds` | number | optional | seconds; expected source depth cadence |

### `stop`

| Field | Type | Presence | Unit / values |
| --- | --- | --- | --- |
| `reason` | string | required | `userStop`, `durationLimit`, `thermalPressure`, `systemPressure`, `appLifecycle`, `storageFailure`, or `captureFailure` |
| `recordedDurationSeconds` | number | required | seconds; finalized recorded duration, `>= 0` |

This contract sets no capture-duration limit.

### `software`

| Field | Type | Presence | Meaning |
| --- | --- | --- | --- |
| `appIdentifier` | string | required | Producer application identifier. |
| `appVersion` | string | required | Producer short version. |
| `buildNumber` | string | required | Producer build number. |
| `schemaWriter` | non-empty string | required | Opaque producer-defined schema-writer identifier. |
