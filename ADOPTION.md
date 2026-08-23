# Producer and Verifier Adoption

This repository is adopted by reference, not as a software dependency.

## TAPCam mobile producer

TAPCamDemo owns capture behavior and the Swift types that serialize manifests,
bindings, and proof envelopes. Its Product Contract decides which artifact
families are current product capabilities. Repository-local module documents
describe packaging, pending signing, Photos export, and local-integrity flow.

Those documents refer here for shared field names, identifiers, byte-layout,
canonicalization, hash, resource-role, and compatibility conventions.

## TAP artifact verifier

Verifier repositories own their parsers, bounded-input handling, local content-
binding recomputation, report model, presentation, and backend calls. They refer
here for the same shared conventions and MUST report implementation gaps rather
than weakening this contract to match a permissive parser.

## No runtime coupling

Neither producer nor verifier imports this repository at runtime. Adoption does
not require a submodule, package-manager dependency, generated binding, schema
loader, or network request. A source repository records the reviewed contract
release or revision in its documentation and maintains its own implementation
and tests.

## Examples and test fixtures

Contract examples and cross-language golden vectors live under
[`examples/`](examples/). A producer or verifier may retain a local copy only
when its tests or runtime genuinely read that copy without requiring this
repository to be checked out. Such a copy is a hermetic mirror, not an
independent authority, and adoption review MUST compare it with the shared
canonical file.

The current TAP Video KLV/zstd v1 vector has three necessary executable mirror
sites:

- TAPCamDemo's `TAPCamDemoTests/TAPVideoManifestTests.swift` loads
  `Docs/Fixtures/TAPVideoManifestV1GoldenVectors.json` by path. Its
  `depthFrame` member mirrors the shared exact vector; the fixture's separate
  manifest object is local decoder input, not shared authority.
- TAPCamDemo's `TAPCamDemoTests/TAPVideoStreamingTests.swift` embeds the same
  raw, zstd1, and complete KLV bytes to prove deterministic producer encoding.
- TAPCamVerifier's `src/video/tapVideo.test.ts` embeds the vector's compressed
  bytes and expected decoded text to exercise its browser decoder without a
  runtime checkout of this repository.

The canonical JSON file's SHA-256 is
`e0d4d2d0d5f199ec942d4b1b7a93c945021cab912418422924faaa43c1fe2cd7`.

The display-transform vector records all eight expected grids. Swift and
Vitest keep their executable orientation tests locally; those tests implement
the shared cases and are not duplicate contract documents.

Generated in-test media used to exercise parsers and ignored physical-device
captures used to exercise platform/container decoding remain local fixtures.
They are executable implementation inputs, not shared format-document copies,
and are not moved here.

## TAP-0094 bootstrap and TAP-0095 deduplication record

- TAPCamDemo adopts this repository from product/module/acceptance documents
  while retaining only producer-local orchestration and executable fixtures.
- TAPCamVerifier adopts this repository from its README and local verification-
  flow document. Its adoption change is isolated from the pre-existing dirty
  runtime working tree used as extraction evidence.
- TAP-0095 makes this repository the single prose authority for signing,
  verification, hash participation, formats, and transport while source
  repositories retain only product, runtime, server, report, and test-runner
  responsibilities.
