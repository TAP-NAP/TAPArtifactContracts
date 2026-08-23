# Extraction Source Snapshot

This file records the evidence boundary for the TAP-0094 initial extraction.
It does not turn implementation code into permanent contract authority.

## TAPCamDemo

- Repository: `TAPCamDemo`
- Git revision before TAP-0094 documentation edits:
  `dc8807e802910ae45e44d6b292a25fa2fed2feda`
- Branch: `main`
- Qualification: the TAP-0094 Board record is an uncommitted documentation
  change on top of this revision.
- Primary sources: `Docs/ProductContract.md`,
  `Docs/LivePhotoBrowserVerification.md`, `Docs/TAPVideoFormatContract.md`,
  `Docs/AppAttest/`, CameraCapture output documentation, current Swift schema
  types, and contract tests.

## TAPCamVerifier

- Repository: `TAPCamVerifier`
- Git revision:
  `20972aff2675cab4a8bb9936bd7fba9115d21951`
- Branch: `main`
- Qualification: the working tree contained pre-existing tracked and untracked
  changes during extraction. It was used as consumer evidence, not silently
  promoted to a clean release baseline.
- Primary evidence: current Rust/WASM Still/Live verifier, TypeScript TAP Video
  verifier, `.tapnap` input routing, and their tests.

## Extraction rule

Where implementation evidence disagrees with the approved TAPCam product or
specialized contract, TAP-0094 records the discrepancy and does not invent a
new wire behavior. No runtime source is modified by this extraction.
