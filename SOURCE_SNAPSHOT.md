# Extraction Source Snapshot

This file records the evidence boundary for the TAP-0094 initial extraction and
TAP-0095 source deduplication. It does not turn implementation code into
permanent contract authority.

## TAPCamDemo

- Repository: `TAPCamDemo`
- Git revision before TAP-0094 documentation edits:
  `dc8807e802910ae45e44d6b292a25fa2fed2feda`
- Branch: `main`
- Qualification at extraction time: the TAP-0094/TAP-0095 Board and
  documentation changes were a documentation-only working tree on top of this
  revision. Their eventual adoption commits are tracked in the source
  repository's Board rather than retroactively changing this evidence snapshot.
- Primary sources: [Product Contract](https://github.com/TAP-NAP/TAPCamDemo/blob/dc8807e802910ae45e44d6b292a25fa2fed2feda/Docs/ProductContract.md),
  [Live Photo browser contract](https://github.com/TAP-NAP/TAPCamDemo/blob/dc8807e802910ae45e44d6b292a25fa2fed2feda/Docs/LivePhotoBrowserVerification.md),
  [TAP Video contract](https://github.com/TAP-NAP/TAPCamDemo/blob/dc8807e802910ae45e44d6b292a25fa2fed2feda/Docs/TAPVideoFormatContract.md),
  [App Attest documents](https://github.com/TAP-NAP/TAPCamDemo/tree/dc8807e802910ae45e44d6b292a25fa2fed2feda/Docs/AppAttest),
  [CameraCapture output documentation](https://github.com/TAP-NAP/TAPCamDemo/blob/dc8807e802910ae45e44d6b292a25fa2fed2feda/TAPCamDemo/CameraCapture/Output/README.md),
  current Swift schema types, and contract tests.
- Shared exact vector source: the `generator` and `depthFrame` portions of the
  [Demo composite vector](https://github.com/TAP-NAP/TAPCamDemo/blob/dc8807e802910ae45e44d6b292a25fa2fed2feda/Docs/Fixtures/TAPVideoManifestV1GoldenVectors.json).
  TAP-0095 preserves that
  composite local fixture because the Swift test target reads it by path. Its
  older manifest object remains local decoder input and is not promoted as a
  normative manifest example.

## TAPCamVerifier

- Repository: `TAPCamVerifier`
- Git revision:
  `20972aff2675cab4a8bb9936bd7fba9115d21951`
- Branch: `main`
- Qualification at extraction time: the working tree contained pre-existing
  tracked and untracked changes. It was used as consumer evidence, not silently
  promoted to a clean release baseline.
- Primary evidence: current Rust/WASM Still/Live verifier, TypeScript TAP Video
  verifier, `.tapnap` input routing, and their tests.

## Extraction rule

Where implementation evidence disagrees with the approved TAPCam product or
specialized contract, TAP-0094 records the discrepancy and does not invent a
new wire behavior. No runtime source is modified by this extraction.
