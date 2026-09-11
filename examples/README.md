# Normative Examples

These synthetic documents show producer fields and exact codec/serialization
cases. `accept`/`reject` applies to the named writer or decoder convention, not
to an artifact signature. Complete verification needs the associated media and
proof under [Capture Binding and Proof](../bindings/capture-binding-and-proof-v1.md).

| Example | Expected document-level outcome | Reason |
| --- | --- | --- |
| [`manifests/still-photo-v1.json`](manifests/still-photo-v1.json) | accept | Still Photo v1 field contract; no `livePhoto`; empty `proofs` |
| [`manifests/live-photo-v1.json`](manifests/live-photo-v1.json) | accept | Live Photo v1 field contract; paired-video declaration; empty `proofs` |
| [`manifests/tap-video-v1.json`](manifests/tap-video-v1.json) | accept | TAP Video v1 field contract; empty `proofs` |
| [`manifests/invalid-photo-nonempty-proofs-v1.json`](manifests/invalid-photo-nonempty-proofs-v1.json) | reject as producer output | Producers leave manifest proofs empty; verification ignores that member and uses only the fixed slot |
| [`transport/still-photo-tapnap-v1.json`](transport/still-photo-tapnap-v1.json) | accept for routing only | One declared primary-photo resource |
| [`transport/live-photo-tapnap-v1.json`](transport/live-photo-tapnap-v1.json) | accept for routing only | One primary-photo and one paired-video resource |
| [`transport/tap-video-tapnap-v1.json`](transport/tap-video-tapnap-v1.json) | accept for routing only | One complete original signed TAP Video MP4 resource |
| [`transport/invalid-tapnap-duplicate-primary-v1.json`](transport/invalid-tapnap-duplicate-primary-v1.json) | reject | Ambiguous duplicate primary-photo resources |
| [`vectors/tap-capture-canonical-json-v1.json`](vectors/tap-capture-canonical-json-v1.json) | accept — exact vector | Immutable canonical UTF-8 bytes for one synthetic object covering ordering, arrays, strings, signed 64-bit integers, and representative binary32/binary64 values |
| [`vectors/tap-video-klv-zstd1-v1-golden-vector.json`](vectors/tap-video-klv-zstd1-v1-golden-vector.json) | accept — exact vector | Raw/zstd1/KLV depth-frame bytes shared across producer and verifier tests; the manifest shape remains the separate manifest example above |
| [`vectors/tap-video-display-transform-v1.json`](vectors/tap-video-display-transform-v1.json) | accept — exact vector | One 3 × 2 top-left-origin grid with expected row-major output for all eight rotation/mirror forms |
| [`vectors/tap-video-extensions-v1.json`](vectors/tap-video-extensions-v1.json) | exact accept/reject cases | Canonical `CALD` and `TAPCAMTELEMETRY1` payload bytes, plus rejected unknown `__proto__` members, UTF-8 BOMs and nonzero Base64 pad bits |

Exact vectors must remain unchanged when implementations adopt them. Local
executable mirrors are permitted only when needed by tests/runtime; compare
those mirrors with these files and document the synchronization check in the
implementation repository.

The canonical-JSON vector's exact UTF-8 case hash is
`47fa943b3af7d6ad7e43653c9175a4d82c7248764dc8b7dbc7b152950d2ab473`.
The KLV/zstd JSON file's SHA-256 is
`e0d4d2d0d5f199ec942d4b1b7a93c945021cab912418422924faaa43c1fe2cd7`.
The display-transform vector records all eight expected grids.

The extension vector has a `context` object and seven entries in `cases`.
`context.durationSeconds`, `context.timeScale`, and
`context.deliveredDepthSampleCount` are sample metadata for a telemetry consumer.
Each case names its `extension` (`cald` or `telemetry`), `expectedDecision`
(`accept` or `reject`), and reason. Decode `utf8Base64` to obtain the exact
extension JSON payload; `utf8ByteCount` and `utf8SHA256` describe those bytes.
The bytes exclude the enclosing KLV record or BMFF UUID box. The `CALD`
positive case stores one synthetic four-byte zero lookup-table value. Its
nonzero-pad-bit negative case decodes to the same value and still fails that
Base64 decoder convention.
