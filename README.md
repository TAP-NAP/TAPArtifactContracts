# TAP Artifact Contracts

`TAPArtifactContracts` is the documentation-only source of truth for artifact
formats shared by the TAPCam mobile producer and TAP artifact verifiers.

It answers three questions:

1. What exact manifest, binding, proof, container, and package conventions does
   TAPCam produce?
2. How does the producer build the hash chain and generate its App Attest
   assertion?
3. What exact conventions must a verifier parse, recompute, compare, reject, or
   forward for server-side assertion verification?

## What lives here

- [Contract index](CONTRACTS.md)
- [Versioning and compatibility](VERSIONING.md)
- [Still Photo](manifests/still-photo-v1.md),
  [Live Photo](manifests/live-photo-v1.md), and
  [TAP Video](manifests/tap-video-v1.md) manifest field contracts
- [Content-binding, signing-binding, proof-envelope, and proof-slot contracts](bindings/capture-binding-and-proof-v1.md)
- [HEIC/JPEG XMP](containers/photo-containers-v1.md) and
  [MP4/KLV](containers/tap-video-container-v1.md) container conventions
- [`.tapnap` routing-sidecar conventions](transport/tapnap-v1.md)
- [Synthetic positive and negative JSON examples plus exact shared golden vectors](examples/README.md)

## What does not live here

This is not a software package. It contains no Mobile App or browser runtime
code, SDK, generated model, parser, validator, or dependency. It does not own
camera selection, capture orchestration, Photos persistence, retry behavior,
Verifier UI, playback, backend deployment, credential operations, or product
claims. It also does not track downstream source snapshots, adoption state,
implementation gaps, or private executable-mirror locations.

The actual per-capture manifest also does not move here:

- Still Photo and Live Photo manifest instances remain in HEIC/JPG XMP at
  `tapdepth:Manifest`.
- TAP Video manifest instances remain in the MP4 top-level manifest `uuid` box.
- App Attest proof envelopes remain in the artifact's fixed proof slot.

This repository records the agreement that those artifacts implement.
Consumers adopt it by reviewed revision, not through a submodule, package
dependency, generated binding, schema loader, or runtime network request.

## Consumer relationship

```text
reviewed TAPArtifactContracts revision
        |                         |
        | documents               | documents
        v                         v
TAPCam mobile producer  --->  artifact  --->  TAP verifier
```

Producer and verifier repositories keep their own implementations. They refer
to the same reviewed contract revision and MUST report any implementation
mismatch without silently redefining this contract.
