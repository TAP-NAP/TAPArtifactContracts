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
| Proof slot | ASCII `TAPCAMPROOFSLOT1` | Fixed 60 KiB proof-slot payload described below |

A reader MUST locate these boxes as top-level boxes by the complete 16-byte
user type. It MUST reject a missing box, a duplicate matching box, a truncated
box, an invalid BMFF length, or a manifest payload larger than 1 MiB. Box order
is not a family identifier; readers route by user type.

The producer finalizes the standard tracks first, then appends the manifest and
an empty proof slot without changing existing media-table offsets. The manifest
and all media bytes remain inside the signed asset-byte view. Only the complete
proof-slot `uuid` box is excluded from that byte view.

## Fixed proof slot v1

The `TAPCAMVIDEOMANF1` box contains no proof body and manifest `proofs` MUST be
empty. The proof envelope is stored only in the `TAPCAMPROOFSLOT1` box.

The proof-slot box payload is exactly `60 * 1024` bytes (61,440 bytes). Its
32-byte header is:

```text
offset   length   encoding       value
0        20       ASCII          TAPCAM-PROOF-SLOT-V1
20       4        bytes          00 00 00 00
24       4        UInt32BE       1
28       4        UInt32BE       proof-envelope byte length N
32       N        UTF-8 JSON     proof envelope
32 + N   remaining bytes         all zero
```

`N` MUST be greater than zero and MUST fit inside the payload after the
32-byte header. Every byte after the envelope MUST be zero. The producer MUST
zero the four reserved bytes. Current readers identify the header from magic
and version but do not independently reject non-zero reserved bytes; this
extraction does not silently add a stricter v1 consumer rule. A reader MUST
reject an incorrect payload size, magic, version, length, JSON envelope, or
padding, as well as missing or duplicate slots.

The current TAP Video content-binding family is
`urn:tapnap:tapcam:video-content-binding:v1`:

```text
assetHash    = SHA-256(all MP4 bytes in file order except exactly the complete
                       TAPCAMPROOFSLOT1 uuid-box byte range)
metadataHash = SHA-256(current canonical JSON bytes of manifest.payload)
```

The excluded range includes the proof box's BMFF header, 16-byte user type, and
entire 60 KiB payload. The manifest box, MP4 metadata, RGB/audio samples, KLV
samples, and every other file byte remain covered. Proof and signing-field
details are defined in
[`../bindings/capture-binding-and-proof-v1.md`](../bindings/capture-binding-and-proof-v1.md).

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

Proof authentication and depth semantics are separate gates:

1. Locate the unique boxes, parse the exact v1 family, and recompute the asset
   and metadata bindings.
2. Authenticate the proof/signing relationship.
3. Only when explicitly requested, scan the actual tracks, KLV records,
   timestamps, calibration indices, and gap coverage.

Untrusted KLV data MUST NOT trigger an unbounded semantic scan before the proof
and fixed byte binding are authenticated. A passing depth scan establishes
container consistency, not physical-world truth.
