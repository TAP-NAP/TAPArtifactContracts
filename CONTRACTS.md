# Contract Index

This page indexes shared artifact obligations. [ProductContract.md](ProductContract.md)
owns product behavior and [BackendContract.md](BackendContract.md) owns shared
HTTP and server trust requirements. All three areas are defined in this
repository; application source and design references are downstream consumers.

**Encoding and decoding** share one definition of each blob's layout, fields,
units, and coordinates. The encoder writes it; the decoder reads it for playback
or analysis. **Signing and verification** cover the bytes actually written:
they use the manifest, resource hashes, and proof to reconstruct the signed
message, without decoding media or judging its values, timing, or quality.

A signed blob can be unusable to a decoder. Verification still requires safe
framing, unambiguous byte coverage, bounded reads, and matching hashes and proof.
Each rule has one owner:

| Area | Normative documents | Responsibility |
| --- | --- | --- |
| Manifest fields | [Still Photo](manifests/still-photo-v1.md), [Live Photo](manifests/live-photo-v1.md), [TAP Video](manifests/tap-video-v1.md) | Artifact metadata, identifiers, units, and resource descriptions |
| Blob encoding and decoding | [Photo containers](containers/photo-containers-v1.md), [Video container and KLV](containers/tap-video-container-v1.md), [Video telemetry](containers/tap-video-capture-telemetry-v1.md) | Shared byte layouts, proof-slot locations, and media interpretation |
| Hashing, signing, and verification | [Capture binding and proof](bindings/capture-binding-and-proof-v1.md) | Covered bytes, hash stages, signed messages, and verification scopes |
| Package routing | [`.tapnap` transport](transport/tapnap-v1.md) | Archive layout and unsigned resource lookup |

Synthetic JSON examples, the exact TAP Video vectors, and their expected
outcomes are indexed in [examples/README.md](examples/README.md).
Reviewed revisions and independent format families are explained in
[VERSIONING.md](VERSIONING.md).

## Terms

- **Artifact manifest**: the signed-metadata input describing one captured
  artifact. It is embedded in the photo or video container.
- **Content binding**: a JSON object containing the artifact and metadata hashes
  plus proof-slot and resource descriptors.
- **Proof envelope**: the App Attest proof JSON stored in the fixed proof slot.
- **Proof slot**: the one fixed-size, locatable container region excluded from
  the artifact-byte hash so the proof can be written after hashing.
- **Routing sidecar**: unsigned `tapcam-export.json` metadata used only to locate
  resources inside `.tapnap`.

The routing sidecar is not the artifact manifest or authenticity evidence.
