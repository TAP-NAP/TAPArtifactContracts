# Still Photo Manifest v1

This document defines the JSON manifest produced for one TAP Still Photo. The
manifest instance remains embedded in the captured HEIC or JPEG; this repository
does not store capture instances.

The accepted synthetic shape example is
[`examples/manifests/still-photo-v1.json`](../examples/manifests/still-photo-v1.json).
It is a manifest example, not a signed-media or canonical-byte golden vector.

## Family identity

The following values identify this family and MUST match exactly.

| Field | Required JSON value |
| --- | --- |
| `schema.id` | `urn:tapnap:tapcam:still-photo-manifest:v1` |
| `schema.version` | integer `1` |
| `schema.mediaType` | `application/vnd.tapnap.still-photo-manifest+json;version=1` |
| `schema.xmpNamespaceURI` | `urn:tapnap:tapcam:depth:1.0` |
| `schema.xmpPrefix` | `tapdepth` |
| `schema.xmpManifestPath` | `tapdepth:Manifest` |
| Canonical payload media type | `application/vnd.tapnap.still-photo-manifest.payload+json;version=1` |

A verifier MUST route on the complete `schema.id`, MUST check every other
`schema` member above, and MUST NOT treat the Live Photo or TAP Video v1 family
as this family.

## JSON and presence rules

The top-level object has exactly the v1 members below. Every
required object member participates in the canonical `payload` hash even when a
member is only descriptive.

| Field | JSON type | Presence and meaning |
| --- | --- | --- |
| `schema` | object | Required; exact family object above. |
| `payload` | object | Required; capture facts defined below. |
| `proofs` | array | Required and MUST be the empty array `[]`. Capture proof data is stored in the fixed proof slot, not here. |

`payload.location` is the sole v1 nullable payload member whose key is
always emitted: it is either a `Location` object or JSON `null`. Other fields
marked optional are omitted when unavailable; the producer does not emit them
as `null`. `payload.livePhoto` MUST be omitted for this family.

All JSON numbers MUST be finite. `integer` means a JSON number with no fractional
part; `resolvedSettingsUniqueID` is a signed 64-bit value and consumers must not
coerce it through a representation that loses integer precision. Pixel counts
and dimensions are in pixels unless a row states another unit. A verifier MUST
validate every emitted payload member, including descriptive values and array
order, and hash the exact raw payload bytes as defined by
[TAP capture canonical JSON](../bindings/capture-binding-and-proof-v1.md#tap-capture-canonical-json).
Descriptive camera, software, time, and location fields are signed claims made
by the producer; they are not independent proof that a real-world fact is true.

## Payload root

| Field | JSON type | Presence | Meaning, unit, vocabulary |
| --- | --- | --- | --- |
| `payload.id` | string | Required | Capture ID; consumers treat it as opaque and compare it byte-for-byte with content-binding and signing IDs. |
| `payload.capturedAt` | string | Required | UTC ISO 8601 internet date-time with fractional seconds. It must equal the binding/proof timestamp. |
| `payload.sessionMode` | string enum | Required | V1 value `singleCam`. |
| `payload.pairingMode` | string enum | Required | `rgbOnly`, `rgbWithApplePairedDepth`, `requiresMultiCam`, or `unsupported`; a normal executable photo-depth plan emits `rgbWithApplePairedDepth`. |
| `payload.alignmentStatus` | string enum | Required | `sameCapturePipeline` or `notCaptured`. |
| `payload.sourceAPIs` | `SourceAPIs` | Required | Apple source API labels. |
| `payload.capture` | `Capture` | Required | Resolved per-shot output facts. |
| `payload.rgbSource` | `RGBSource` | Required | Requested RGB source facts. |
| `payload.depthSource` | `DepthSourceSelection` | Required | Requested and resolved depth-source facts. |
| `payload.pairing` | `Pairing` | Required | Pairing decision and release eligibility. |
| `payload.zoom` | `Zoom` | Required | Requested/resolved raw video zoom facts. Zoom factors are unitless. |
| `payload.crop` | `Crop` | Required | Preview crop metadata; it does not assert a destructive image crop. |
| `payload.resolvedSession` | `ResolvedSession` | Required | Actual configured capture session/device. |
| `payload.selectedDepthCamera` | `SelectedDepthCamera` | Required | Selected depth row and resolved device display facts. |
| `payload.selectedZoom` | `SelectedZoom` | Required | Selected zoom row. |
| `payload.photoLens` | `PhotoLens` | Required | Requested lens/FOV label and resolved camera facts. |
| `payload.depthBackend` | `DepthBackendSelection` | Required | Requested and resolved depth backend. |
| `payload.camera` | `Camera` | Required | AVFoundation device and format snapshot. |
| `payload.photo` | `Photo` | Required | Encoded photo dimensions/orientation and source metadata-key inventory. |
| `payload.depth` | `Depth` | Required | Auxiliary depth/disparity availability, encoding, source, and optional calibration. |
| `payload.alignment` | `Alignment` | Required | Relationship between depth and the primary image. |
| `payload.location` | `Location` or `null` | Required key | Location snapshot when allowed and available; otherwise explicit `null`. |
| `payload.software` | `Software` | Required | Producing app identity/version snapshot. |
| `payload.livePhoto` | object | Forbidden | MUST be omitted. Its presence belongs only to Live Photo manifest v1. |

## Source and capture objects

### `SourceAPIs`

All four fields are required strings and a v1 producer emits the exact
values below. They are descriptive labels, not executable API instructions.

| Field | Exact v1 value |
| --- | --- |
| `photo` | `AVCapturePhotoOutput / AVCapturePhoto` |
| `depth` | `AVCapturePhoto.depthData / AVDepthData` |
| `camera` | `AVCaptureDevice / AVCaptureDevice.Format` |
| `location` | `CLLocationManager.requestLocation / CLLocation` |

### `Capture`

| Field | JSON type | Presence | Meaning or vocabulary |
| --- | --- | --- | --- |
| `resolvedSettingsUniqueID` | integer | Required | Signed 64-bit AVFoundation resolved-settings identifier; no physical unit. |
| `requestedCodec` | string enum | Required | AVFoundation codec raw value: `hvc1` for the reviewed HEIC profile or `jpeg` for the reviewed JPEG profile. It must match the actual container. |
| `depthDataDeliveryEnabled` | boolean | Required | V1 producer value `true`. |
| `embedsDepthDataInPhoto` | boolean | Required | V1 producer value `true`; per-shot depth may still be unavailable. |
| `depthDataFiltered` | boolean | Required | V1 producer value `true`. |
| `depthAvailability` | string enum | Required | `available` or `unavailable`; must equal `payload.depth.availability`. |
| `photoQualityPrioritization` | string enum | Required | `speed`, `balanced`, or `quality`; the v1 producer value is `quality`. This is AVFoundation prioritization, not a size, resolution, or compression guarantee. |

### `RGBSource`

| Field | JSON type | Presence | Meaning or vocabulary |
| --- | --- | --- | --- |
| `id` | string | Required | Producer profile/device identifier; opaque. |
| `displayName` | string | Required | Producer display label. |
| `deviceType` | string | Required | `AVCaptureDevice.DeviceType.rawValue`; platform vocabulary, not a closed TAP enum. |
| `deviceName` | string | Required | AVFoundation localized device name. |
| `position` | string enum | Required | `front`, `back`, `unspecified`, or `unknown`. |
| `sourceKind` | string enum | Required | `physical`, `virtual`, or `depthVirtual`. |
| `requestedReferenceZoomFactor` | number | Required | Unitless raw reference zoom factor. |

### `DepthSourceSelection`

| Field | JSON type | Presence | Meaning or vocabulary |
| --- | --- | --- | --- |
| `selectionMode` | string enum | Required | `auto`, `manual`, or `debugDepthOverride`. |
| `requestedDepthSourceID` | string | Optional, omitted | Requested depth-row identifier. |
| `requestedDepthSourceDisplayName` | string | Optional, omitted | Requested depth-row display label. |
| `requestedDepthSourceKind` | string enum | Optional, omitted | `lidarDepth`, `trueDepth`, `dualCameraDisparity`, `dualWideDisparity`, or `portraitSemanticDepth`. |
| `compatibilityStatus` | string enum | Required | `compatible`, `requiresMultiCam`, `unsupportedFormat`, `unsupportedZoom`, `releasePackagingUnsupported`, or `unavailable`. |
| `compatibilityReason` | string | Optional, omitted | Producer diagnostic reason; no closed vocabulary. |
| `resolvedDeviceID` | string | Optional, omitted | Resolved AVFoundation device ID. |
| `resolvedDeviceType` | string | Optional, omitted | Resolved `AVCaptureDevice.DeviceType.rawValue`. |
| `resolvedDeviceName` | string | Optional, omitted | Resolved localized device name. |

### `Pairing`

| Field | JSON type | Presence | Meaning or vocabulary |
| --- | --- | --- | --- |
| `mode` | string enum | Required | Same pairing vocabulary as `payload.pairingMode`. |
| `status` | string enum | Required | Same compatibility vocabulary as `depthSource.compatibilityStatus`. |
| `requiresMultiCam` | boolean | Required | Whether the planned pairing mode requires MultiCam. |
| `releaseAllowed` | boolean | Required | Whether the producing plan passed its photo-depth capture gate. |
| `alignmentStatus` | string enum | Required | `sameCapturePipeline` or `notCaptured`. |

## Zoom, crop, and session objects

### `Zoom`

| Field | JSON type | Presence | Meaning or vocabulary |
| --- | --- | --- | --- |
| `requestedZoomID` | string | Optional, omitted | Requested zoom-profile identifier. |
| `requestedZoomFactor` | number | Optional, omitted | Unitless requested raw `videoZoomFactor`. |
| `actualVideoZoomFactor` | number | Optional, omitted | Unitless raw `AVCaptureDevice.videoZoomFactor` chosen by the plan. |
| `depthSafeRanges` | array of `ZoomRange` | Required | Ordered list of inclusive, unitless depth-safe raw zoom ranges; may be empty. |
| `isContinuous` | boolean | Required | Producer capability classification. |
| `isDiscrete` | boolean | Required | Producer capability classification. |

Each `ZoomRange` has required numeric `lowerBound` and `upperBound` members.

### `Crop`

| Field | JSON type | Presence | Meaning or vocabulary |
| --- | --- | --- | --- |
| `mode` | string enum | Required | Exact v1 value `previewOnly`. |
| `cropRectNormalized` | object | Required | Required numeric `x`, `y`, `width`, and `height`. Components are normalized to `[0,1]` in AVFoundation metadata-output coordinates, origin upper-left, positive x right, positive y down. The producer clamps each component independently. |
| `destructiveFinalCropApplied` | boolean | Required | Exact v1 value `false`. |
| `sourceAPI` | string | Required | Exact v1 value `AVCaptureVideoPreviewLayer.metadataOutputRectConverted(fromLayerRect:)`. |

### `ResolvedSession`

| Field | JSON type | Presence | Meaning or vocabulary |
| --- | --- | --- | --- |
| `mode` | string enum | Required | Exact v1 value `singleCam`. |
| `resolvedCaptureDeviceID` | string | Required | Opaque AVFoundation device ID. |
| `resolvedCaptureDeviceType` | string | Required | `AVCaptureDevice.DeviceType.rawValue`. |
| `resolvedCaptureDeviceName` | string | Required | Localized device name. |
| `activePrimaryConstituentDeviceType` | string | Optional, omitted | Active physical constituent type when the resolved device is virtual. |
| `activePrimaryConstituentDeviceName` | string | Optional, omitted | Active constituent localized name. |

### `SelectedDepthCamera`

All members are required. `id`, `displayName`, `deviceType`, `deviceName`, and
`position` are strings. `position` uses `front`, `back`, `unspecified`, or
`unknown`. If no requested depth row exists, the producer uses `id: "none"` and
`displayName: "None"`; device fields still describe the resolved capture device.

### `SelectedZoom`

All members are required: string `id`, string `displayName`, and numeric,
unitless `zoomFactor`. Producer fallbacks are `zoom-unknown`, `unknown`, and
`1.0` when no selected zoom is recorded.

## Lens, backend, and camera objects

### `PhotoLens`

| Field | JSON type | Presence | Meaning or unit |
| --- | --- | --- | --- |
| `requestedLensID` | string | Required | Opaque requested RGB/lens identifier. |
| `requestedDisplayName` | string | Required | Requested source display label. |
| `requestedFocalLengthLabel` | string | Required | Producer-formatted FOV label such as `24mm`; descriptive. |
| `labelSource` | string | Required | Producer provenance label for the FOV calculation; no closed wire enum. |
| `requestedZoomFactor` | number | Required | Unitless raw zoom factor, fallback `1.0`. |
| `requestedReferenceZoomFactor` | number | Required | Unitless reference zoom factor. |
| `requestedEquivalentFocalLength35mmMillimeters` | number | Optional, omitted | Requested 35 mm-equivalent focal length in millimetres. |
| `position` | string enum | Required | `front`, `back`, `unspecified`, or `unknown`. |
| `resolvedCaptureDeviceType` | string | Required | `AVCaptureDevice.DeviceType.rawValue`. |
| `resolvedCaptureDeviceName` | string | Required | Localized device name. |
| `resolvedActivePrimaryConstituentDeviceType` | string | Optional, omitted | Active constituent type. |
| `resolvedActivePrimaryConstituentDeviceName` | string | Optional, omitted | Active constituent localized name. |

### `DepthBackendSelection`

| Field | JSON type | Presence | Meaning or vocabulary |
| --- | --- | --- | --- |
| `selectionMode` | string enum | Required | `auto`, `manual`, or `debugDepthOverride`. |
| `requestedBackendID` | string | Optional, omitted | Requested depth backend ID. |
| `requestedBackendDisplayName` | string | Optional, omitted | Requested backend display label. |
| `resolvedBackendID` | string | Required | Resolved depth-row ID, falling back to the capture-device ID. |
| `resolvedBackendDisplayName` | string | Required | Resolved depth-row label, falling back to the capture-device name. |
| `resolvedCaptureDeviceType` | string | Required | `AVCaptureDevice.DeviceType.rawValue`. |
| `resolvedCaptureDeviceName` | string | Required | Localized device name. |
| `actualVideoZoomFactor` | number | Required | Unitless raw zoom factor, fallback `1.0`. |

### `Camera`

| Field | JSON type | Presence | Meaning or unit |
| --- | --- | --- | --- |
| `localizedName` | string | Required | AVFoundation localized device name. |
| `uniqueID` | string | Required | AVFoundation device identifier; opaque and potentially sensitive. |
| `modelID` | string | Required | AVFoundation device model identifier; potentially sensitive. |
| `deviceType` | string | Required | `AVCaptureDevice.DeviceType.rawValue`. |
| `position` | string enum | Required | `front`, `back`, `unspecified`, or `unknown`. |
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

## Photo, depth, alignment, location, and software

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
| `availability` | string enum | Required | `available` or `unavailable`; must equal `capture.depthAvailability`. |
| `auxiliaryDataKind` | string enum | Required | `depth`, `disparity`, or `none`. |
| `depthDataType` | string | Required | AVDepthData FourCC as four printable ASCII characters or `0x` plus eight uppercase hexadecimal digits; `none` when unavailable. |
| `metricUnit` | string enum | Required | `meters` for native depth, `convertDisparityToDepthMeters` for disparity, or `none` when unavailable. |
| `conversionPath` | string enum | Required | `nativeDepthMeters`, `AVDepthData.converting(toDepthDataType: kCVPixelFormatType_DepthFloat32)`, or `depthUnavailable`. |
| `width` | integer | Required | Depth/disparity map width in pixels; `0` when unavailable. |
| `height` | integer | Required | Depth/disparity map height in pixels; `0` when unavailable. |
| `pixelFormat` | string | Required | Depth map pixel-format FourCC/hex string; `none` when unavailable. |
| `orientation` | string enum | Required | `appleAuxiliaryDepthNative` or `unavailable`. |
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

### `Alignment`

`depthToImage` is a required string: `appleAuxiliaryDepthNative` when depth is
available, otherwise `unavailable`. The former means the auxiliary depth map
uses Apple's image-aligned auxiliary-depth convention; it does not add a new
TAP transform.

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

## Examples and expected decision

The accepted shape and rejected non-empty-proof example are indexed with their
expected decisions in [`examples/README.md`](../examples/README.md).

## Required consistency checks

A conforming Still Photo verifier MUST, before treating the manifest as bound:

1. require the exact Still Photo family object and omitted `payload.livePhoto`;
2. require `proofs: []`;
3. parse the manifest from the XMP location in
   [photo-containers-v1.md](../containers/photo-containers-v1.md);
4. require `capture.depthAvailability == depth.availability` and compare actual
   auxiliary depth/disparity presence with that value;
5. extract, validate, and hash the exact raw `payload` value under
   [capture-binding-and-proof-v1.md](../bindings/capture-binding-and-proof-v1.md);
6. require the content-binding family
   `urn:tapnap:tapcam:still-photo-content-binding:v1`.
