# TAP Video Capture Telemetry v1

Status: adopted optional v1 extension

Capture telemetry records Apple depth-filtering observations and bounded Core
Motion samples for one TAP Video. It is optional in the existing TAP Video v1
container and is a separate family from its manifest, depth KLV and proof.
It describes device motion; it does not supply camera translation, full 6DoF
pose, scene flow, confidence, or evidence that a scene is true.

## Container and binding

The extension is one optional top-level BMFF `uuid` box with the 16 ASCII-byte
user type `TAPCAMTELEMETRY1`. Its payload is UTF-8 JSON using
[TAP capture canonical JSON](../bindings/capture-binding-and-proof-v1.md#tap-capture-canonical-json),
at most 4 MiB. A producer appends it before computing the content binding;
it MUST NOT alter media tables or add a track. A telemetry decoder rejects
duplicate matching boxes as ambiguous.

The box is part of the MP4's existing byte binding. Unknown or malformed
telemetry makes that interpretation unavailable. An absent box means unknown
provenance, not that filtering was off or motion unavailable.

## JSON fields

Every object below has exactly the listed keys. All keys are required;
`motionToCaptureOffsetSeconds` alone permits explicit `null`. Missing keys,
unknown keys and duplicate keys fail. Counts are non-negative integers within
the exact safe-integer range `0...9007199254740991`; measured numbers are finite.

The [shared extension vectors](../examples/vectors/tap-video-extensions-v1.json)
include exact accepted bytes and rejected unknown-member and UTF-8 BOM cases.

The root keys are `schema`, `filtering`, and `motion`.

| `schema` field | Required value |
| --- | --- |
| `id` | `urn:tapnap:tapcam:video-capture-telemetry:v1` |
| `version` | integer `1` |
| `mediaType` | `application/vnd.tapnap.video-capture-telemetry+json;version=1` |

| `filtering` field | Type / meaning |
| --- | --- |
| `requestedEnabled` | boolean; the Apple depth-filtering request frozen for this recording |
| `filteredSampleCount` | count of delivered depth samples whose actual `AVDepthData.isDepthDataFiltered` was true |
| `unfilteredSampleCount` | count of delivered depth samples whose actual `AVDepthData.isDepthDataFiltered` was false |

The two counts record delivered observations, including samples subsequently
lost to encoding or metadata append. They need not match the requested switch.
They count delivered depth samples, not stored frames or RGB frames. With no
delivered depth, both are zero and no actual filtering result can be inferred.

| `motion` field | Type / meaning |
| --- | --- |
| `status` | `available`, `unavailable`, `noSamples`, or `partial`, defined below |
| `referenceFrame` | exact string `xArbitraryZVertical` |
| `deviceCoordinateSystem` | exact string `core-motion-device-right-handed` |
| `timeBase` | exact string `capture-relative-seconds` |
| `motionToCaptureOffsetSeconds` | finite number or explicit `null`; first retained sample's `ptsSeconds` minus its original Core Motion timestamp, seconds; diagnostic only |
| `sampleIntervalSeconds` | requested update interval, seconds, `0 < value <= 1`; current producer requests `1/30` |
| `droppedSampleCount` | producer-recorded dropped update count |
| `errorCount` | observed motion acquisition or clock-conversion errors; excludes the ordinary absence of hardware support |
| `samples` | ordered array of at most 8,192 sample objects |

| Sample field | Type / unit / order |
| --- | --- |
| `ptsSeconds` | finite number, seconds relative to the first recorded RGB presentation timestamp |
| `quaternion` | four finite numbers `[x,y,z,w]`, the unmodified `CMAttitude.quaternion` relative to the selected reference frame; dimensionless |
| `rotationRate` | three finite numbers `[x,y,z]`, radians per second, device axes |
| `gravity` | three finite numbers `[x,y,z]`, multiples of standard gravity `g`, device axes |
| `userAcceleration` | three finite numbers `[x,y,z]`, multiples of `g`, device axes; the gravity-removed acceleration estimate |

Device axes are fixed to the device in portrait: positive x toward its right
edge, positive y toward its top, and positive z out of the screen toward the
user. They form a right-handed system. The reference frame has vertical z and
an arbitrary horizontal heading. These are Core Motion device/reference axes,
not the RGB pixel grid, the camera optical frame, or a geographic heading.
Do not rotate these stored values for UI orientation, front-camera mirroring,
or the signed RGB display transform. A consumer that later needs camera-frame
motion must explicitly account for that mapping. No translation is recorded.

## Clock, bounds and availability

For every motion update, the producer maps the Core Motion monotonic host-time
timestamp through the capture session's synchronization clock, then subtracts
the first recorded RGB PTS. It MUST NOT substitute callback arrival time, wall
clock, or an assumed equality of host and capture clocks. The first observed
offset is diagnostic; clocks can differ in rate, so it MUST NOT be used as a
constant mapping for the remaining samples. A consumer uses each stored PTS.

- `available`: the producer reports available observations.
- `partial`: the producer reports incomplete observations.
- `noSamples`: the producer reports no retained samples.
- `unavailable`: acquisition, reference frame, or clock mapping was unavailable.

Collection starts for recording and stops on stop, error or lifecycle
termination. Reaching 8,192 samples stops retaining additional motion data,
increments dropped count for discarded updates and keeps recording RGB/depth.
Producers preserve whole, valid JSON; they MUST NOT truncate the encoded box.
If encoding would exceed 4 MiB, discard retained tail samples, include those in
the dropped count and recompute status before serialization.

Smoothing settings and derived display frames are not part of this extension.
The observation counts and motion samples remain unchanged when a player
changes its display policy.
