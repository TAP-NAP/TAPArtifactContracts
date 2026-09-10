# TAP Video MP4 Container and Timed Depth v1

Status: v1 container contract
Manifest: [`../manifests/tap-video-v1.md`](../manifests/tap-video-v1.md)

One TAP Video capture is one `video/mp4` file. It is not a ZIP archive, a
`.tapnap` package, a JSON sidecar, or a set of synchronized files. The MP4 owns
the standard playable tracks, the embedded TAP Video manifest, the fixed proof
slot, and—when samples were stored—the TAP-private timed-depth track.

## Track composition

| Part | Cardinality | V1 contract |
| --- | --- | --- |
| RGB video | exactly one | Standard MP4 video track. The manifest binds the actual codec and track facts. |
| Audio | zero or one | Standard audio track. The manifest binds the actual codec and distinguishes `captured`, `notCaptured`, and `unavailable`. |
| TAP timed depth | zero or one | Private timed-metadata track, present only when at least one real depth sample was stored. |

With stored depth, the file therefore has one RGB track, one metadata track,
and optionally one audio track. With zero stored depth, it has one RGB track,
optionally one audio track, and no TAP metadata track. `container.trackCount`
and every manifest track fact MUST agree with the finalized MP4. Track IDs MUST
be distinct.

Ordinary MP4 players may ignore the private metadata and top-level `uuid`
boxes and play RGB/audio normally.

## Top-level TAP boxes

The file contains exactly one of each top-level BMFF `uuid` box:

| Purpose | 16-byte user type | Payload |
| --- | --- | --- |
| Manifest | ASCII `TAPCAMVIDEOMANF1` | UTF-8 TAP Video manifest v1 JSON; at most 1 MiB |
| Proof slot | ASCII `TAPCAMPROOFSLOT1` | Common fixed [61,440-byte proof-slot payload](../bindings/capture-binding-and-proof-v1.md#proofslot) |

A reader MUST locate these boxes as top-level boxes by the complete 16-byte
user type. It MUST reject a missing box, a duplicate matching box, a truncated
box, an invalid BMFF length, or a manifest payload larger than 1 MiB. Box order
is not a family identifier; readers route by user type.

The producer finalizes the standard tracks first, then appends the manifest and
an empty proof slot without changing existing media-table offsets. The manifest
and all media bytes remain inside the signed asset-byte view. Only the complete
proof-slot `uuid` box is excluded from that byte view.

The optional independently versioned
[capture-telemetry extension](tap-video-capture-telemetry-v1.md) adds a top-level
UUID box covered by that same asset-byte view. It changes no v1 track or KLV
record and grants no additional proof-slot exclusion.

The `TAPCAMVIDEOMANF1` box contains no proof body and manifest `proofs` MUST be
empty. Its proof envelope is stored only in the proof-slot box. The exact TAP
Video family, excluded range, hash participation, and signing fields are defined
in [Capture Binding and Proof v1](../bindings/capture-binding-and-proof-v1.md).

## TAP timed-depth metadata track

When `depthCoverage.sampleCount > 0`, the metadata item identifier is exactly:

```text
mdta/com.tapnap.depth.klv
```

The item data type is raw data. The finalized MP4 metadata sample-entry codec
recorded in `depthCoverage.trackCodec` is `mebx`. Each timed item carries one
independently decodable TAP KLV frame. The MP4 timed-metadata sample timestamp
and the KLV `PTS ` timestamp describe the same capture-relative instant within
one tick of the finer relevant timescale.

This is a TAP-private schema that borrows a compact KLV pattern. It does not
adopt GoPro/GPMF field meanings and is not a standard depth-video track.

### Serialized `mebx` sample wrapper

TAPCam supplies the KLV bytes as one AVFoundation metadata-item value.
AVFoundation serializes the raw MP4 `mebx` sample as a QuickTime timed-metadata
atom; the 8-byte prefix is not part of `TAPDepthKLVFrame`:

```text
offset   length   encoding       meaning
0        4        UInt32BE       complete metadata-value atom size
4        4        UInt32BE       local_key_id
8        size-8   bytes          TAP KLV frame
```

Each v1 metadata sample contains one item, so the atom size equals the complete
sample size.
`local_key_id` is neither a TAP version nor an arbitrary non-zero flag. It is a
track-local identifier whose metadata-key-table entry in the `mebx` sample
description MUST resolve to namespace `mdta` and key
`com.tapnap.depth.klv`. A reader MUST parse that mapping and MUST NOT assume the
identifier is always `1`. Values `0` and `0xFFFFFFFF` are reserved by the
QuickTime timed-metadata format and are not valid mappings for the TAP item.

A reader rejects a truncated atom, a size below 8, a size that escapes the
sample, extra v1-family items, or a missing/ambiguous/wrong key mapping.
Only after removing this AVFoundation-owned atom prefix does it apply the TAP
KLV rules below. See Apple's
[timed metadata sample data format](https://developer.apple.com/documentation/quicktime-file-format/timed_metadata_sample_data_format)
and [metadata key atom](https://developer.apple.com/documentation/quicktime-file-format/metadata_key_atom).

## KLV frame version 1

The raw value of one timed metadata item is a sequence of records:

```text
fourCC[4] | payloadLengthUInt32BE[4] | payload[payloadLength] |
zero padding to the next 4-byte boundary
```

The FourCC is four ASCII bytes. The payload length and all integer control
values below are big-endian. Alignment padding is zero. Readers identify records
by key, not by position; record order is not significant.

| Key | Presence | Payload | Meaning |
| --- | --- | --- | --- |
| `TVER` | required, once | 4-byte UInt32BE | KLV frame schema version; MUST equal `1`. |
| `FRAM` | required, once | 4-byte UInt32BE | Zero-based timed-depth MP4 sample ordinal. Values MUST be contiguous in sample order from `0` through `depthCoverage.sampleCount - 1`. |
| `PTS ` | required, once | 8-byte Int64BE value followed by 4-byte Int32BE timescale | Capture-relative presentation time in ticks; timescale MUST be `> 0`; seconds are `value / timescale`. |
| `COMP` | required, once | ASCII bytes | `raw`, `lzfse`, or `zstd1`. The value MUST also be permitted by the manifest `compressionPolicy`. |
| `ULEN` | required, once | 4-byte UInt32BE | Uncompressed packed-frame byte count. It MUST equal `depthCoverage.format.uncompressedFrameByteCount`. |
| `CALI` | optional, at most once | 4-byte UInt32BE | Zero-based index into `spatialRegistration.calibrationTable`; it MUST be in bounds. |
| `DPTH` | required, once | bytes | Raw or independently compressed packed depth/disparity bytes. |

Unknown four-character keys are skippable for forward-compatible parsing. An
unknown key acquires no v1 semantics and MUST NOT replace a required key.

The FourCC values `DKND`, `PIXF`, `DIM `, `RSTR`, and `CALR` are not v1 frame
records. Their invariant facts live in the manifest; a writer MUST NOT emit
them as substitutes for required records, and a reader MUST NOT assign them v1
semantics.

### Inline calibration extension candidate: `CALD`

Status: local, unpublished compatibility candidate; no new reviewed commit pin.
This candidate uses the existing skippable-key rule without changing `TVER=1`,
the manifest family, or any hash exclusion. It preserves the current frame's
calibration when the bounded table cannot supply a `CALI` index; calibration
values can change on every captured frame.

`CALD` is optional, at most once, and mutually exclusive with `CALI`. Its payload
is at most **3,072 bytes** of UTF-8 canonical JSON containing exactly these keys:

| Field | Type | Meaning |
| --- | --- | --- |
| `calibration` | `CameraCalibration` | The current frame's full AVFoundation calibration, using the existing [field definitions](../manifests/tap-video-v1.md#cameracalibration), units and coordinates. |
| `schemaVersion` | integer | Exactly `1`. |

Canonical serialization uses sorted object keys and no insignificant whitespace;
duplicate keys, unknown keys, malformed UTF-8, non-finite numbers, and trailing
bytes are rejected. The calibration has the five existing required fields and
only the two existing optional lookup-table fields, each omitted, `null`, or a
canonical padded-base64 string. Matrices contain exactly 9 and 12 finite numbers;
dimensions, pixel size, and distortion center retain their existing finite-number
constraints. `Dimensions` and `Point` retain their exact existing field sets.
Measurement numbers retain the existing numeric-token rules; `schemaVersion`
uses the integer token `1`, not `1.0`.

A reader recognizing this candidate validates `CALD` even when it does not
render 3D. It rejects simultaneous `CALI`/`CALD`, invalid calibration, and an
oversized payload. Existing readers may skip `CALD` as an unknown key and
continue their v1 checks. Other unknown keys retain the same skippable behavior.

The 16-entry table and all `calibrationCoverage` counters remain unchanged:
`indexedSampleCount` counts only `CALI`; a frame with calibration that could not
enter the full table remains `overflowUnindexedSampleCount` even when it carries
`CALD`. Missing calibration is not fabricated, and inline calibration does not
create a table index. Old recordings without a stored per-frame calibration
cannot recover that calibration from this extension.

The `CALD` bytes remain in the timed-depth payload and therefore in the existing
signed asset-byte view. No excluded range, proof slot, resource, track, or
transport family is added.

## Packed frame and codec rules

- Accepted stored formats are Float16 or Float32 depth/disparity:
  `hdep`, `fdep`, `hdis`, and `fdis`.
- `DPTH` stores exactly the logical row bytes. Capture-buffer padding is not
  part of the packed frame.
- Packed sample bytes use the manifest byte order, exactly `little-endian` in
  v1.
- Frames are encoded independently. `COMP` and the manifest compression policy
  select `raw`, `lzfse`, or Zstandard level 1 (`zstd1`). V1 consumers MUST
  implement all three values and accept each only when the signed manifest
  policy permits it.
- After decoding, the byte count MUST equal both `ULEN` and the manifest
  `uncompressedFrameByteCount`. Depth samples are not quantized, synthesized,
  interpolated, or duplicated to match RGB cadence.

## Bounds and rejection

The v1 KLV/container limits are:

| Limit | Maximum |
| --- | --- |
| Manifest-box JSON payload | 1 MiB |
| Records in one KLV frame | 32 |
| Uncompressed packed depth frame | 32 MiB |
| Encoded `DPTH` payload | 32 MiB |
| Complete KLV record sequence for one frame | 32 MiB + 4,096 bytes |
| Manifest depth-gap entries | 1,024 |
| Manifest calibration-table entries | 16 |

This format sets no capture-duration limit. Consumers may impose additional
bounded-input limits, such as a maximum file size, box count, or sample count;
those safety budgets do not change the v1 wire format.

A KLV reader MUST fail closed for:

- more than 32 records or a complete KLV frame above its bound;
- a truncated header, payload, or alignment region;
- a non-ASCII FourCC or non-zero alignment padding;
- a duplicate key, including an unknown duplicate key;
- a missing required key or a required control field of the wrong length;
- `TVER` other than `1`, a non-positive `PTS ` timescale, or an unsupported
  `COMP` value;
- a `CALI` index outside the signed table;
- a frame index, sample count, timestamp, track fact, or codec policy that does
  not match the manifest; or
- decoded bytes whose count differs from `ULEN` or the signed format.

The producer and authenticated reader MUST preserve real missing intervals as
signed manifest gaps. They MUST NOT fabricate KLV samples to conceal drops.

## Verification order

Apply the shared local and backend gates in
[Capture Binding and Proof v1](../bindings/capture-binding-and-proof-v1.md).
Untrusted KLV MUST NOT trigger an unbounded semantic scan before the local
binding gate passes. A bounded semantic scan may then run while the backend is
pending, but its result MUST remain untrusted and MUST NOT produce a final
authenticated verdict until the backend gate also passes. It establishes
container consistency, not App Attest authenticity or physical-world truth.
