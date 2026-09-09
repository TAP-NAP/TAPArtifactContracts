# TAPCam Documentation

Product requirements, artifact formats, and design rationale for TAPCam.
These documents describe how captures are represented, signed, transported,
and verified, together with the app behavior that supports that process.

```text
capture -> manifest + content binding -> App Attest proof -> signed artifact
       -> Photos / transport -> verifier checks binding and proof -> result
```

## Core documents

- [Product contract](ProductContract.md): required behavior, lifecycle,
  claim boundaries, and non-goals.
- [Artifact contract index](CONTRACTS.md): manifest, binding, proof, container,
  KLV, and transport conventions shared by producers and verifiers.
- [App Attest backend contract](BackendContract.md): registration, HTTP, server
  trust, and replay boundaries.
- [Planes design](PlanesTechnicalDesign.md): the photo geometry algorithm and
  its reasoning.

Manifest families are independent:
[Still Photo](manifests/still-photo-v1.md),
[Live Photo](manifests/live-photo-v1.md), and
[TAP Video](manifests/tap-video-v1.md).
The [versioning policy](VERSIONING.md) defines compatibility and revision rules.
[Synthetic examples and exact vectors](examples/README.md) show field values,
serialization, and expected validation results.

Actual per-capture manifests are embedded in their media: Still/Live in
HEIC/JPG XMP and Video in the MP4 manifest UUID box. App Attest proofs occupy
the artifact's fixed proof slot.

## Using these documents

Producer and verifier implementations adopt the shared wire contracts at a
specific reviewed revision. Their repositories provide source navigation,
build instructions, tests, and implementation coverage. This documentation
repository contains no runtime library or software dependency.

[Acceptance procedures](Acceptance.md) describe how to validate native behavior,
visual parity, and device performance by capability.
