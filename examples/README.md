# Normative Examples

These files make field names, omission rules, family routing, rejection
conditions, and TAP Video KLV/decompression bytes concrete. All values are
synthetic.

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
| `manifests/still-photo-v1.json` | accept | Current Still Photo v1 field contract; no `livePhoto`; empty `proofs` |
| `manifests/live-photo-v1.json` | accept | Current Live Photo v1 field contract; paired-video declaration; empty `proofs` |
| `manifests/tap-video-v1.json` | accept | Current TAP Video v1 field contract; empty `proofs` |
| `manifests/invalid-photo-nonempty-proofs-v1.json` | reject | Manifest proof bodies are forbidden; the proof belongs in the fixed slot |
| `transport/still-photo-tapnap-v1.json` | accept for routing only | One declared primary-photo resource |
| `transport/live-photo-tapnap-v1.json` | accept for routing only | One primary-photo and one paired-video resource |
| `transport/invalid-tapnap-duplicate-primary-v1.json` | reject | Ambiguous duplicate primary-photo resources |
| `vectors/tap-video-klv-zstd1-v1-golden-vector.json` | accept — exact vector | Raw/zstd1/KLV depth-frame bytes shared across producer and verifier tests; the current manifest shape remains the separate manifest example above |
| `vectors/tap-video-display-transform-v1.json` | accept — exact vector | One 3 × 2 top-left-origin grid with expected row-major output for all eight rotation/mirror forms |

The transport examples describe `tapcam-export.json`, which is unsigned routing
metadata and never authenticity evidence.

The TAP Video KLV/zstd vector is different from the shape examples: its byte
counts, hash, base64 payloads, and KLV record bytes are exact. Consumers MUST NOT
regenerate it merely to make an incompatible implementation pass. Source
repositories may keep byte-identical or literal hermetic mirrors when their
test runners cannot depend on another checkout; those mirrors are not a second
contract authority.
