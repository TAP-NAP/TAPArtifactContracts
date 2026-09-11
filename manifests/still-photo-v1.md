# Still Photo Manifest v1

This document defines the producer's manifest fields, embedded in one HEIC/JPEG.
[Photo Containers](../containers/photo-containers-v1.md) defines their placement;
[Capture Binding and Proof](../bindings/capture-binding-and-proof-v1.md) defines
hashing and verification. See the [synthetic example](../examples/manifests/still-photo-v1.json).

## Family identity

Producers write the following family and encoding values.

| Field | Required JSON value |
| --- | --- |
| `schema.id` | `urn:tapnap:tapcam:still-photo-manifest:v1` |
| `schema.version` | integer `1` |
| `schema.mediaType` | `application/vnd.tapnap.still-photo-manifest+json;version=1` |
| `schema.xmpNamespaceURI` | `urn:tapnap:tapcam:depth:1.0` |
| `schema.xmpPrefix` | `tapdepth` |
| `schema.xmpManifestPath` | `tapdepth:Manifest` |

## JSON and presence rules

Producers write these top-level members:

| Field | JSON type | Presence and meaning |
| --- | --- | --- |
| `schema` | object | Required; exact family object above. |
| `payload` | object | Required; capture facts defined below. |
| `proofs` | array | Producer writes `[]`. |

`payload.location` is the sole v1 nullable payload member whose key is
always emitted: it is either a `Location` object or JSON `null`. Other fields
marked optional are omitted when unavailable; the producer does not emit them
as `null`. `payload.livePhoto` MUST be omitted for this family.

`integer` describes a JSON number with no fractional part. Pixel counts and
dimensions are in pixels unless stated otherwise. Encoding uses
[TAP capture canonical JSON](../bindings/capture-binding-and-proof-v1.md#tap-capture-canonical-json).

## Payload root

| Field | JSON type | Presence | Meaning, unit, vocabulary |
| --- | --- | --- | --- |
| `payload.id` | string | Required | Capture ID; consumers treat it as opaque and compare it byte-for-byte with content-binding and signing IDs. |
| `payload.capturedAt` | string | Required | UTC ISO 8601 internet date-time with fractional seconds. It equals `contentDigest.capturedAt`; the excluded outer proof timestamp is only a producer copy. |
| `payload.camera` | `Camera` | Required | AVFoundation device and format snapshot. |
| `payload.photo` | `Photo` | Required | Encoded photo dimensions/orientation and source metadata-key inventory. |
| `payload.depth` | `Depth` | Required | Auxiliary depth/disparity availability, encoding, source, and optional calibration. |
| `payload.location` | `Location` or `null` | Required key | Location snapshot when allowed and available; otherwise explicit `null`. |
| `payload.software` | `Software` | Required | Producing app identity/version snapshot. |
| `payload.livePhoto` | object | Forbidden | MUST be omitted. Its presence belongs only to Live Photo manifest v1. |

## Capture data

### `Camera`

| Field | JSON type | Presence | Meaning or unit |
| --- | --- | --- | --- |
| `localizedName` | string | Required | AVFoundation localized device name. |
| `uniqueID` | string | Required | AVFoundation device identifier; opaque and potentially sensitive. |
| `modelID` | string | Required | AVFoundation device model identifier; potentially sensitive. |
| `deviceType` | string | Required | `AVCaptureDevice.DeviceType.rawValue`. |
| `position` | string enum | Required | Actual capture-device facing: `front`, `back`, `unspecified`, or `unknown`. |
| `activePrimaryConstituentDeviceType` | string | Optional, omitted | Active constituent type. |
| `activePrimaryConstituentDeviceName` | string | Optional, omitted | Active constituent localized name. |
| `activeFormat` | `CameraFormat` | Required | Active RGB/video format. |
| `activeDepthFormat` | `CameraFormat` | Optional, omitted | Active depth format. |
| `lensPosition` | number | Optional, omitted | Unitless AVFoundation lens position, normally in `[0,1]`; direction/meaning follows AVFoundation. |
| `minimumFocusDistanceMillimeters` | integer | Optional, omitted | Minimum focus distance in millimetres; emitted only when positive. |
| `nominalFocalLengthIn35mmFilmMillimeters` | number | Optional, omitted | Positive 35 mm-equivalent focal length in millimetres when the OS exposes it. |

Each `CameraFormat` contains required string `mediaSubType` (printable FourCC or
`0x` plus eight uppercase hexadecimal digits), required integer `width` and
`height` in pixels, and optional numeric `maxFrameRate` in frames per second.

### `Photo`

| Field | JSON type | Presence | Meaning |
| --- | --- | --- | --- |
| `width` | integer | Required | Resolved primary photo width in pixels. |
| `height` | integer | Required | Resolved primary photo height in pixels. |
| `orientation` | string | Required | `cgImagePropertyOrientation:<n>` using the source CGImagePropertyOrientation integer, or `unspecified`. |
| `metadataKeys` | array of string | Required | Lexicographically sorted top-level keys from `AVCapturePhoto.metadata`; array order is signed. |

### `Depth`

| Field | JSON type | Presence | Meaning, unit, vocabulary |
| --- | --- | --- | --- |
| `availability` | string enum | Required | Producer observation: `available` or `unavailable`. |
| `auxiliaryDataKind` | string enum | Required | `depth`, `disparity`, or `none`. |
| `depthDataType` | string | Required | AVDepthData FourCC as four printable ASCII characters or `0x` plus eight uppercase hexadecimal digits; `none` when unavailable. |
| `metricUnit` | string enum | Required | `meters` for native depth, `convertDisparityToDepthMeters` for disparity, or `none` when unavailable. |
| `conversionPath` | string enum | Required | `nativeDepthMeters`, `AVDepthData.converting(toDepthDataType: kCVPixelFormatType_DepthFloat32)`, or `depthUnavailable`. |
| `width` | integer | Required | Depth/disparity map width in pixels; `0` when unavailable. |
| `height` | integer | Required | Depth/disparity map height in pixels; `0` when unavailable. |
| `pixelFormat` | string | Required | Depth map pixel-format FourCC/hex string; `none` when unavailable. |
| `orientation` | string enum | Required | `appleAuxiliaryDepthNative` means Apple's primary-image-aligned auxiliary-depth convention; otherwise `unavailable`. |
| `accuracy` | string enum | Required | `relative`, `absolute`, `unknown`, or `unavailable`. |
| `quality` | string enum | Required | `low`, `high`, `unknown`, or `unavailable`. |
| `isFiltered` | boolean | Required | `AVDepthData.isDepthDataFiltered`; producer emits `false` when unavailable. |
| `source` | `DepthSource` | Required | Device-based source classification. |
| `cameraCalibration` | `CameraCalibration` | Optional, omitted | Calibration snapshot when supplied by `AVDepthData`. |

`DepthSource` has four required strings:

- `captureDeviceType`: `AVCaptureDevice.DeviceType.rawValue`;
- `captureDeviceName`: localized device name;
- `sensingMethod`: `lidarDepthCamera`, `trueDepthCamera`,
  `multiCameraStereoOrComputational`, or
  `singleCameraComputationalOrUnknown`;
- `lidarParticipation`: `explicit`, `notApplicable`, or `notAsserted`.

`CameraCalibration` has the required fields below. It copies
`AVCameraCalibrationData` without changing its coordinate convention.

| Field | JSON type | Meaning, unit, coordinate/order |
| --- | --- | --- |
| `intrinsicMatrixReferenceWidth` | number | Intrinsic reference-frame width in pixels. |
| `intrinsicMatrixReferenceHeight` | number | Intrinsic reference-frame height in pixels. |
| `pixelSizeMillimeters` | number | Size of one reference-dimension pixel in millimetres. |
| `lensDistortionLookupTablePresent` | boolean | Whether the forward radial-distortion table was present. The table bytes are not in the manifest. |
| `inverseLensDistortionLookupTablePresent` | boolean | Whether the inverse radial-distortion table was present. The table bytes are not in the manifest. |
| `lensDistortionCenterX` | number | Pixel x offset from the upper-left of the intrinsic reference frame. |
| `lensDistortionCenterY` | number | Pixel y offset from the upper-left of the intrinsic reference frame. |
| `intrinsicMatrix` | array of 9 numbers | 3 x 3 camera intrinsic K matrix in pixels, flattened by SIMD columns: `[c0.x,c0.y,c0.z,c1.x,...,c2.z]`. Principal-point origin is upper-left. |
| `extrinsicMatrix` | array of 12 numbers | 3 x 4 camera-to-world `[R|t]` pose, flattened by four SIMD columns of three values. Rotation is unitless; translation is millimetres; pose is relative to the reference camera. |

### `Location`

When not `null`, all fields are required:

| Field | JSON type | Meaning and unit |
| --- | --- | --- |
| `latitude` | number | WGS 84 latitude in decimal degrees. |
| `longitude` | number | WGS 84 longitude in decimal degrees. |
| `altitude` | number | `CLLocation.altitude` in metres. |
| `horizontalAccuracy` | number | `CLLocation.horizontalAccuracy` in metres. |
| `verticalAccuracy` | number | `CLLocation.verticalAccuracy` in metres. |
| `timestamp` | string | UTC ISO 8601 timestamp with fractional seconds. |

Location is signed metadata and can be sensitive. Its presence is not proof that
the location is physically true.

### `Software`

Required string members are `appName`, `bundleIdentifier`, `version`, and
`build`. They are copied from the producing app bundle, with producer fallback
strings where bundle metadata is unavailable. They are descriptive and may be
sensitive in non-production builds.
