# Contract Index

This page indexes shared artifact obligations. [ProductContract.md](ProductContract.md)
owns product behavior and [BackendContract.md](BackendContract.md) owns shared
HTTP and server trust requirements. All three areas are defined in this
repository; application source and design references are downstream consumers.

| Area | Normative document | Responsibility |
| --- | --- | --- |
| Still Photo manifest | [`manifests/still-photo-v1.md`](manifests/still-photo-v1.md) | JSON fields, XMP identity, omission rules |
| Live Photo manifest | [`manifests/live-photo-v1.md`](manifests/live-photo-v1.md) | JSON fields and paired-resource declaration |
| TAP Video manifest | [`manifests/tap-video-v1.md`](manifests/tap-video-v1.md) | JSON fields and manifest box identity |
| Capture binding and proof | [`bindings/capture-binding-and-proof-v1.md`](bindings/capture-binding-and-proof-v1.md) | Canonical bytes, family hash participation, producer signing, local reconstruction, App Attest assertion verification, signed resources |
| Photo containers | [`containers/photo-containers-v1.md`](containers/photo-containers-v1.md) | XMP discovery and HEIC/JPEG proof-slot layout |
| TAP Video container and KLV | [`containers/tap-video-container-v1.md`](containers/tap-video-container-v1.md) | MP4 boxes and KLV v1 records, including optional inline calibration `CALD` |
| TAP Video capture telemetry | [`containers/tap-video-capture-telemetry-v1.md`](containers/tap-video-capture-telemetry-v1.md) | Optional independently versioned UUID, filtering observations and bounded device-motion samples |
| `.tapnap` transport | [`transport/tapnap-v1.md`](transport/tapnap-v1.md) | Archive and unsigned routing-sidecar conventions |

Synthetic JSON examples, the exact TAP Video vectors, and their expected
outcomes are indexed in [examples/README.md](examples/README.md).
`CALD` and `TAPCAMTELEMETRY1` are adopted optional v1 extensions. Their exact
acceptance and rejection vectors share the same authority as their field and
byte definitions.

## Terms

- **Artifact manifest**: the signed-metadata input describing one captured
  artifact. It is embedded in the photo or video container.
- **Manifest contract**: the versioned field and serialization agreement in
  this repository.
- **Content binding**: a JSON object containing the artifact and metadata hashes
  plus proof-slot and resource descriptors.
- **Proof envelope**: the App Attest proof JSON stored in the fixed proof slot.
- **Proof slot**: the one fixed-size, locatable container region excluded from
  the artifact-byte hash so the proof can be written after hashing.
- **Routing sidecar**: unsigned `tapcam-export.json` metadata used only to locate
  resources inside `.tapnap`.

These terms are not interchangeable. In particular, the routing sidecar is not
the artifact manifest and is not authenticity evidence.
