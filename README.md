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
- Still Photo, Live Photo, and TAP Video manifest field contracts
- Content-binding, signing-binding, proof-envelope, and proof-slot contracts
- HEIC/JPEG XMP and MP4/KLV container conventions
- `.tapnap` routing-sidecar conventions
- Synthetic positive and negative JSON examples plus exact shared golden vectors
- [Extraction source snapshot](SOURCE_SNAPSHOT.md)
- [Known extraction divergences](KNOWN_DIVERGENCES.md)
- [Producer and verifier adoption boundary](ADOPTION.md)

## What does not live here

This is not a software package. It contains no Mobile App or browser runtime
code, SDK, generated model, parser, validator, or dependency. It does not own
camera selection, capture orchestration, Photos persistence, retry behavior,
Verifier UI, playback, backend deployment, credential operations, or product
claims.

The actual per-capture manifest also does not move here:

- Still Photo and Live Photo manifest instances remain in HEIC/JPG XMP at
  `tapdepth:Manifest`.
- TAP Video manifest instances remain in the MP4 top-level manifest `uuid` box.
- App Attest proof envelopes remain in the artifact's fixed proof slot.

This repository records the agreement that those artifacts implement.

## Consumer relationship

```text
TAPArtifactContracts release
        |                         |
        | documents               | documents
        v                         v
TAPCam mobile producer  --->  artifact  --->  TAP verifier
```

Producer and verifier repositories keep their own implementations. They refer
to the same released contract revision and report any implementation mismatch
without silently redefining this contract.

## Current scope

The initial extraction is tracked as `TAP-0094` in TAPCamDemo. `TAP-0095`
extends the same documentation boundary with the explicit producer-signing,
verifier-reconstruction, App Attest verification, hash-participation, and
example-ownership rules, then removes duplicate source-repository prose. Both
tasks preserve the current, distinct v1 bytes and behavior. See
[SOURCE_SNAPSHOT.md](SOURCE_SNAPSHOT.md) for the source revisions and known
working-tree qualifications used during extraction.
