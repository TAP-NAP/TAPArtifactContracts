# Contract Index

This page separates shared artifact obligations from product- or
implementation-local behavior.

| Area | Normative document | Owner here | Explicitly not owned here |
| --- | --- | --- | --- |
| Still Photo manifest | `manifests/still-photo-v1.md` | JSON fields, XMP identity, omission rules | Camera runtime and Photos export |
| Live Photo manifest | `manifests/live-photo-v1.md` | JSON fields and paired-resource declaration | Live Photo capture/playback UX |
| TAP Video manifest | `manifests/tap-video-v1.md` | JSON fields and manifest box identity | Recorder, pending queue, playback |
| Capture binding and proof | `bindings/capture-binding-and-proof-v1.md` | Canonical bytes, hashes, signed resources, signing relationship | App Attest key lifecycle and server deployment |
| Photo containers | `containers/photo-containers-v1.md` | XMP discovery and HEIC/JPEG proof-slot layout | ImageIO/Photos implementation |
| TAP Video container and KLV | `containers/tap-video-container-v1.md` | MP4 boxes and KLV v1 records | AVFoundation writer/reader implementation |
| `.tapnap` transport | `transport/tapnap-v1.md` | Archive and unsigned routing-sidecar conventions | Share UI and temporary-file lifecycle |

Synthetic JSON examples and their expected document-level outcomes are indexed
in [examples/README.md](examples/README.md).

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
