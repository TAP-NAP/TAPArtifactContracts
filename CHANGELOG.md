# Changelog

## Unreleased — TAP-0095 source deduplication

- Specify the producer's ordered content-binding and App Attest signing steps.
- Specify local reconstruction separately from backend cryptographic assertion
  verification.
- Add a Still/Live/Video hash-participation matrix.
- Adopt the TAP Video v1 KLV/zstd depth-frame vector as the shared canonical
  copy while allowing only necessary hermetic test mirrors in source
  repositories. Keep the current manifest example separate from an older local
  decoder fixture that is not normative.
- Define the QuickTime `mebx` sample wrapper/key-table relationship and add an
  all-eight-direction TAP Video display-transform vector.
- Replace duplicate source-repository format prose with references to this
  repository without changing runtime behavior or wire bytes.

## Unreleased — TAP-0094 initial extraction

- Establish the documentation-only repository boundary.
- Record current Still Photo, Live Photo, and TAP Video artifact conventions.
- Record current binding, proof, container, KLV, and `.tapnap` conventions.
- Add synthetic normative examples and explicit rejection cases.

This extraction does not change any format identifier, serialized byte,
embedding location, producer behavior, or verifier behavior.
