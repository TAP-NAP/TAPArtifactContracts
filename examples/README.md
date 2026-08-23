# Normative Examples

These files make field names, omission rules, family routing, and rejection
conditions concrete. All values are synthetic.

They are documentation examples, not captured media or executable conformance
fixtures. A manifest marked `accept` means its JSON shape and family-level
semantics conform to the documented manifest contract. It does not claim that a
standalone JSON file has a valid artifact hash or App Attest assertion; complete
artifact verification also requires the associated HEIC/JPEG/MP4 bytes and
proof slot.

| Example | Expected document-level outcome | Reason |
| --- | --- | --- |
| `manifests/still-photo-v1.json` | accept | Current Still Photo v1 field contract; no `livePhoto`; empty `proofs` |
| `manifests/live-photo-v1.json` | accept | Current Live Photo v1 field contract; paired-video declaration; empty `proofs` |
| `manifests/tap-video-v1.json` | accept | Current TAP Video v1 field contract; empty `proofs` |
| `manifests/invalid-photo-nonempty-proofs-v1.json` | reject | Manifest proof bodies are forbidden; the proof belongs in the fixed slot |
| `transport/still-photo-tapnap-v1.json` | accept for routing only | One declared primary-photo resource |
| `transport/live-photo-tapnap-v1.json` | accept for routing only | One primary-photo and one paired-video resource |
| `transport/invalid-tapnap-duplicate-primary-v1.json` | reject | Ambiguous duplicate primary-photo resources |

The transport examples describe `tapcam-export.json`, which is unsigned routing
metadata and never authenticity evidence.
