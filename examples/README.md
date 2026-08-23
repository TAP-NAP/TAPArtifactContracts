# Normative Examples

These files make field names, omission rules, family routing, rejection
conditions, canonical JSON bytes, and TAP Video KLV/decompression bytes
concrete. All values are synthetic.

They are documentation examples, not captured media or executable conformance
fixtures. A manifest marked `accept` means its JSON shape and family-level
semantics conform to the documented manifest contract. It does not claim that a
standalone JSON file has a valid artifact hash or App Attest assertion; complete
artifact verification also requires the associated HEIC/JPEG/MP4 bytes and
proof slot.
A vector marked `accept — exact vector` is a positive byte or transform oracle
for the named sub-contract, not a complete independently verifiable artifact.

| Example | Expected document-level outcome | Reason |
| --- | --- | --- |
| [`manifests/still-photo-v1.json`](manifests/still-photo-v1.json) | accept | Still Photo v1 field contract; no `livePhoto`; empty `proofs` |
| [`manifests/live-photo-v1.json`](manifests/live-photo-v1.json) | accept | Live Photo v1 field contract; paired-video declaration; empty `proofs` |
| [`manifests/tap-video-v1.json`](manifests/tap-video-v1.json) | accept | TAP Video v1 field contract; empty `proofs` |
| [`manifests/invalid-photo-nonempty-proofs-v1.json`](manifests/invalid-photo-nonempty-proofs-v1.json) | reject | Manifest proof bodies are forbidden; the proof belongs in the fixed slot |
| [`transport/still-photo-tapnap-v1.json`](transport/still-photo-tapnap-v1.json) | accept for routing only | One declared primary-photo resource |
| [`transport/live-photo-tapnap-v1.json`](transport/live-photo-tapnap-v1.json) | accept for routing only | One primary-photo and one paired-video resource |
| [`transport/invalid-tapnap-duplicate-primary-v1.json`](transport/invalid-tapnap-duplicate-primary-v1.json) | reject | Ambiguous duplicate primary-photo resources |
| [`vectors/tap-capture-canonical-json-v1.json`](vectors/tap-capture-canonical-json-v1.json) | accept — exact vector | Immutable canonical UTF-8 bytes for one synthetic object covering ordering, arrays, strings, signed 64-bit integers, and representative binary32/binary64 values |
| [`vectors/tap-video-klv-zstd1-v1-golden-vector.json`](vectors/tap-video-klv-zstd1-v1-golden-vector.json) | accept — exact vector | Raw/zstd1/KLV depth-frame bytes shared across producer and verifier tests; the manifest shape remains the separate manifest example above |
| [`vectors/tap-video-display-transform-v1.json`](vectors/tap-video-display-transform-v1.json) | accept — exact vector | One 3 × 2 top-left-origin grid with expected row-major output for all eight rotation/mirror forms |

The transport examples describe `tapcam-export.json`, which is unsigned routing
metadata and never authenticity evidence.

The canonical-JSON and TAP Video KLV/zstd vectors differ from the shape examples:
their byte counts, hashes, and encoded bytes are exact. Consumers MUST NOT
regenerate them merely to make an incompatible implementation pass. Source
repositories may keep byte-identical or literal hermetic mirrors only when
their tests or runtime genuinely read the local copy and cannot depend on
another checkout. Those mirrors are not a second contract authority. Adoption
review MUST compare each mirror with its shared canonical file.

Downstream repositories own the locations and synchronization checks for any
necessary executable mirrors. This repository deliberately does not index or
link to those private copies.

The canonical-JSON vector's exact UTF-8 case hash is
`47fa943b3af7d6ad7e43653c9175a4d82c7248764dc8b7dbc7b152950d2ab473`.
The KLV/zstd JSON file's SHA-256 is
`e0d4d2d0d5f199ec942d4b1b7a93c945021cab912418422924faaa43c1fe2cd7`.
The display-transform vector records all eight expected grids.
