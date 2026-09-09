# TAPCam Documentation

This repository is the main source for TAPCam product requirements, artifact
contracts, and design rationale. Read only the boundary relevant to your work.
Implementation and visual prototypes remain in their own repositories.

```text
capture -> manifest + content binding -> App Attest proof -> signed artifact
       -> Photos / transport -> verifier checks binding and proof -> result
```

## Core documents

- [Product contract](ProductContract.md): behavior, lifecycle, settled decisions,
  claim boundaries, and non-goals. Requirements do not imply every path ships.
- [Artifact contract index](CONTRACTS.md): exact manifest, binding, proof,
  container, KLV, and transport conventions shared by producers and verifiers.
- [App Attest backend contract](BackendContract.md): registration, HTTP, server
  trust, and replay boundaries.
- [Planes design](PlanesTechnicalDesign.md): the photo geometry algorithm and
  its reasoning.

Manifest families are independent:
[Still Photo](manifests/still-photo-v1.md),
[Live Photo](manifests/live-photo-v1.md), and
[TAP Video](manifests/tap-video-v1.md).
The [versioning policy](VERSIONING.md) governs changes; [synthetic examples and
exact vectors](examples/README.md) make the wire conventions concrete.

Actual per-capture manifests stay inside their media: Still/Live in HEIC/JPG XMP,
Video in the MP4 manifest UUID box, and App Attest proofs in the fixed proof slot.
The prototype's manifest stays beside its executable visual fixtures; it records
visual revisions and approvals, not artifact bytes or product requirements.

## Repository boundary

These documents define requirements without requiring another repository's
agent guide or task board. Code repositories keep their own source map, run/test
commands, and implementation coverage. They adopt shared wire contracts by a
reviewed revision; there is no package, submodule, generated binding, or runtime
network dependency on this repository.

Keep code, parsers, SDKs, real media, credentials, and backend deployment out of
this repository. A source revision may support an implementation claim; it cannot
replace a missing requirement in these documents. A documentation move alone
does not change a consumer's pinned wire revision.

## When needed

[Acceptance procedures](Acceptance.md) are grouped by capability and read only
for the selected validation. [Work notes](ProjectBoard.md) retain unfinished
work and legacy IDs for continuity. They are optional; routine fixes require no
new Task, full backlog scan, or per-task Markdown.
