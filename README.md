# TAP Artifact Contracts

## Purpose

This repository is the normative source for TAP product behavior, artifact
formats, and shared backend interfaces. It defines how captures are represented,
signed, transported, and verified, together with the app behavior and trust
boundaries that support that process. Implementations consume these requirements;
their source, documentation, and visual references do not redefine them.

```text
capture -> manifest + content binding -> App Attest proof -> signed artifact
       -> Photos / transport -> verifier checks binding and proof -> result
```

## Usage

Read the product contract for behavior, then the artifact index and backend
contract for the interfaces your implementation consumes:

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
serialization, and expected validation results, including the adopted optional
TAP Video inline-calibration and capture-telemetry extensions.

Actual per-capture manifests are embedded in their media: Still/Live in
HEIC/JPG XMP and Video in the MP4 manifest UUID box. App Attest proofs occupy
the artifact's fixed proof slot.

This is a documentation repository: there is no application to build, package to
install, or runtime service to start. Review Markdown directly. When editing,
check relative links and anchors and parse the synthetic JSON examples; validate
exact vectors if their owning contract is affected. Follow the versioning policy
before changing any producer byte or consumer decision.

Producer and verifier implementations adopt the shared wire contracts at a
specific reviewed revision. Their repositories provide source navigation,
build instructions, tests, and implementation coverage. This documentation
repository contains no runtime library or software dependency.

[Acceptance procedures](Acceptance.md) describe how to validate native behavior,
visual parity, and device performance by capability.

## Principles

- Keep each requirement in its owning contract; use local links for related rules.
- Keep artifact integrity, depth assessment, and real-world claims distinct.
- Preserve exact family identifiers, hash inputs, proof slots, and fail-closed
  decisions; editorial changes do not advance consumer pins.
- Distinguish required behavior from implementation coverage and measured evidence.

## Repository structure

| Path | Responsibility |
| --- | --- |
| `ProductContract.md` | Product scope, user-visible behavior, lifecycle, and claim boundaries |
| `BackendContract.md` | App Attest registration, HTTP interfaces, and server trust boundaries |
| `CONTRACTS.md`, `VERSIONING.md` | Contract navigation, family identity, revision and compatibility policy |
| `manifests/` | Still Photo, Live Photo, and TAP Video metadata fields |
| `bindings/` | Canonical bytes, capture hashing, proof, and verification gates |
| `containers/` | Photo/MP4 embedding, proof slots, timed depth, and capture telemetry |
| `transport/` | `.tapnap` packaging and unsigned resource routing |
| `examples/` | Synthetic field examples and exact byte/transform vectors |
| `PlanesTechnicalDesign.md` | Photo-local geometry algorithm and its limits |
| `Acceptance.md` | Capability-specific procedures and evidence requirements |

## Repository dependencies

The dependency direction is from implementations to this repository. TAPCamDemo
consumes product and producer requirements; TAPCamVerifier consumes transport,
artifact, proof, and verification requirements; App Attest clients and backends
consume the shared HTTP and trust requirements. Each implementation records the
reviewed contract revision it adopts and documents its own build and deployment.
No application, component, design, or project-management repository is required
to read or define these contracts. Apple platform references explain external
APIs where cited; they do not replace TAP-specific requirements.
