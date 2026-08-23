# TAP Video MP4 Container and Timed Depth v1

Status: current v1 container contract
Manifest: [`../manifests/tap-video-v1.md`](../manifests/tap-video-v1.md)

One TAP Video capture is one `video/mp4` file. It is not a ZIP archive, a
`.tapnap` package, a JSON sidecar, or a set of synchronized files. The MP4 owns
the standard playable tracks, the embedded TAP Video manifest, the fixed proof
slot, and—when samples were stored—the TAP-private timed-depth track.

## Track composition

| Part | Cardinality | Current contract |
| --- | --- | --- |
| RGB video | exactly one | Standard MP4 video track. Runtime currently falls back to H.264, but the manifest binds the actual codec and track facts. |
| Audio | zero or one | Standard audio track; current captured audio uses AAC. The manifest distinguishes `captured`, `notCaptured`, and `unavailable`. |
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

For the current one-item sample, the atom size equals the complete sample size.
`local_key_id` is neither a TAP version nor an arbitrary non-zero flag. It is a
track-local identifier whose metadata-key-table entry in the `mebx` sample
description MUST resolve to namespace `mdta` and key
`com.tapnap.depth.klv`. A reader MUST parse that mapping and MUST NOT assume the
identifier is always `1`. Values `0` and `0xFFFFFFFF` are reserved by the
QuickTime timed-metadata format and are not valid mappings for the TAP item.

A reader rejects a truncated atom, a size below 8, a size that escapes the
sample, extra current-family items, or a missing/ambiguous/wrong key mapping.
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
values below are big-endian. Alignment padding is zero. Records may appear in
the current producer order shown below, but readers identify records by key,
not by position.

| Key | Presence | Payload | Meaning |
| --- | --- | --- | --- |
| `TVER` | required, once | 4-byte UInt32BE | KLV frame schema version; MUST equal `1`. |
| `FRAM` | required, once | 4-byte UInt32BE | Zero-based stored-depth frame index. Current streams use contiguous indices in sample order. |
| `PTS ` | required, once | 8-byte Int64BE value followed by 4-byte Int32BE timescale | Capture-relative presentation time in ticks; timescale MUST be `> 0`; seconds are `value / timescale`. |
| `COMP` | required, once | ASCII bytes | `raw`, `lzfse`, or `zstd1`. The value MUST also be permitted by the manifest `compressionPolicy`. |
| `ULEN` | required, once | 4-byte UInt32BE | Uncompressed packed-frame byte count. It MUST equal `depthCoverage.format.uncompressedFrameByteCount`. |
| `CALI` | optional, at most once | 4-byte UInt32BE | Zero-based index into `spatialRegistration.calibrationTable`; it MUST be in bounds. |
| `DPTH` | required, once | bytes | Raw or independently compressed packed depth/disparity bytes. |

The current producer emits records in this order: `TVER`, `FRAM`, `PTS `,
`COMP`, `ULEN`, optional `CALI`, then `DPTH`. Unknown four-character keys are
skippable for forward-compatible parsing. An unknown key acquires no v1
semantics and MUST NOT replace a required key.

The schema source also defines FourCC constants `DKND`, `PIXF`, `DIM `, `RSTR`,
and `CALR`, but the current v1 frame writer does not emit them. Their invariant
facts live in the manifest, so these constants are not additional required v1
records and this extraction does not assign them new wire meanings.

## Packed frame and codec rules

- Accepted stored formats are Float16 or Float32 depth/disparity:
  `hdep`, `fdep`, `hdis`, and `fdis`.
- `DPTH` stores exactly the logical row bytes. Capture-buffer padding is not
  part of the packed frame.
- Packed sample bytes use the manifest byte order, currently `little-endian`.
- Frames are encoded independently. The current writer prefers Zstandard level
  1 (`zstd1`) and falls back to `raw` if compression fails or is not smaller.
- Current v1 readers accept `raw`, `lzfse`, and `zstd1`. LZFSE remains readable
  compatibility behavior; it is not a claim that the current writer emits it.
- After decoding, the byte count MUST equal both `ULEN` and the manifest
  `uncompressedFrameByteCount`. Depth samples are not quantized, synthesized,
  interpolated, or duplicated to match RGB cadence.

## Bounds and rejection

The versioned KLV/container limits extracted from current producers and readers
are:

| Limit | Maximum |
| --- | --- |
| Manifest-box JSON payload | 1 MiB |
| Records in one KLV frame | 32 |
| Uncompressed packed depth frame | 32 MiB |
| `DPTH` encoded payload accepted by the current frame codec | 32 MiB |
| Complete KLV record sequence for one frame | 32 MiB + 4,096 bytes |
| Manifest depth-gap entries | 1,024 |
| Manifest calibration-table entries | 16 |

The 180-second capture default is not a format or decoder limit. Individual
consumers may impose additional bounded-input limits, such as a browser's
maximum file size, box count, or sample count; those safety budgets do not
change the v1 wire format.

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
