# TAP Artifact Contracts

This repository defines TAP product behavior, artifact formats and shared
backend interfaces. Implementations consume these requirements; their source
and visual references do not redefine them.

## Reading path

| Document | Read it for |
| --- | --- |
| [Contract index](CONTRACTS.md) | Manifest fields, shared encoder/decoder layouts, hashes and signatures |
| [Product contract](ProductContract.md) | App behavior, lifecycle and claim boundaries |
| [Backend contract](BackendContract.md) | App Attest registration, HTTP, trust and replay policy |
| [Planes design](PlanesTechnicalDesign.md) | Photo geometry and its limits |
| [Acceptance](Acceptance.md) | Capability-specific procedures and measured evidence |
| [Examples and vectors](examples/README.md) | Synthetic fields and exact byte/transform cases |
| [Versioning](VERSIONING.md) | Independent format families and reviewed revisions |

TAPCamDemo primarily produces artifacts. TAPCamVerifier primarily reads their
blobs and reconstructs their signed hashes. Native playback and export checks
reuse the same format and hash definitions. App Attest assertion verification
belongs to the backend.

## Editing and adoption

This is a documentation repository with no runtime to build or install. Check
local Markdown links/anchors, parse the synthetic JSON, and check exact vectors
when their owning format changes. Do not regenerate vectors to match an
implementation. Keep media, credentials and application code in their owning
repositories.

Each implementation pins the exact reviewed contract commit and documents its
own source, builds, tests and deployment. That Git revision is separate from
wire-format versions. Follow [VERSIONING.md](VERSIONING.md) for behavior or byte
changes; implementation coverage requires source and measured evidence.
