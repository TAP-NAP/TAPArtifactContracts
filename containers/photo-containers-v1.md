# Photo Containers v1

Task: `TAP-0094`. Source boundary: [SOURCE_SNAPSHOT.md](../SOURCE_SNAPSHOT.md).

This document defines where Still Photo v1 and Live Photo v1 manifests and
proofs live in reviewed HEIC and JPEG photo-depth artifacts. It does not mandate
an ImageIO implementation and it does not move the manifest into a sidecar.

## Artifact map

```text
HEIC primary photo                         JPEG primary photo
------------------                         ------------------
primary image + Apple auxiliary depth      SOI
primary-image XMP metadata                 APP11 TAP proof slot
  tapdepth:Manifest                          (producer places it immediately here)
top-level uuid TAP proof-slot box           metadata segments, including XMP APP1
                                             tapdepth:Manifest
                                           image data / EOI
```

For a Live Photo, the primary photo uses the same layout and the original paired
resource is a separate `paired-video.mov`. The MOV contains neither this photo
manifest nor the photo proof slot; its complete bytes are named in the Live
Photo content binding.

## XMP manifest

The manifest is the UTF-8 JSON string value of the XMP property identified by:

| Item | Exact value |
| --- | --- |
| Namespace URI | `urn:tapnap:tapcam:depth:1.0` |
| Preferred prefix | `tapdepth` |
| Property/path | `tapdepth:Manifest` |

The JSON string is the complete top-level manifest object, including `schema`,
`payload`, and `proofs: []`. A verifier MUST resolve the XMP property, undo XML
entity encoding, parse the resulting JSON, and require exactly one manifest
property. XML attribute-style and element-style representations of the same
namespace/property are container serializations, not different manifest
families.

For HEIC, the property is XMP metadata associated with the primary image. Its
physical HEIF metadata-item offsets are not fixed by this contract; discovery is
by the XMP namespace and property, not a hard-coded box offset. For JPEG, the
property is in the JPEG XMP metadata carried by APP1; the APP1 byte location may
vary after metadata-preserving ImageIO output.

The producer obtains the base file from
`AVCapturePhoto.fileDataRepresentation(with:)`, injects XMP by copying the image
source with metadata merge rather than decoding/re-encoding pixels, and requires
exact JSON-string readback before reserving the proof slot. A verifier uses the
resulting bytes and MUST NOT reconstruct the signed artifact from decoded image
pixels.

The producer also writes this exact EXIF UserComment pointer in both reviewed
containers:

```text
TAPDepthHEIC/1; metadata=xmp:tapdepth:Manifest
```

It is a discovery hint only. It is not the manifest, a family identifier, a
content hash, or proof evidence.

## Fixed proof-slot payload

Both container forms carry the same fixed 61,440-byte payload:

| Payload-relative byte range | Size | Encoding and meaning |
| --- | ---: | --- |
| `0..<20` | 20 | ASCII `TAPCAM-PROOF-SLOT-V1` |
| `20..<24` | 4 | Producer-reserved zero bytes |
| `24..<28` | 4 | Unsigned 32-bit big-endian version, exact value `1` |
| `28..<32` | 4 | Unsigned 32-bit big-endian proof-envelope byte length |
| `32..<(32+length)` | `length` | Canonical proof JSON envelope bytes |
| `(32+length)..<61440` | remainder | Zero padding |

Maximum envelope length is `61440 - 32 = 61408` bytes. An unsigned pending
artifact has length `0`; reading a signed proof from such a slot reports a
missing envelope. A signed artifact requires length greater than zero. Bytes
after a non-empty envelope MUST all be zero. The producer initializes the four
reserved bytes to zero; current readers identify the header from magic and
version and do not independently reject non-zero reserved bytes.

The hash exclusion is the complete enclosing UUID box or APP11 segment. The
61,440-byte payload alone is not the excluded range.

## HEIC/BMFF slot

The HEIC slot is one top-level ISO BMFF `uuid` box with 16-byte user type:

```text
hex:   54 41 50 43 41 4d 50 52 4f 4f 46 53 4c 4f 54 31
ASCII: TAPCAMPROOFSLOT1
```

Current producer layout for a normal 32-bit-size box is:

| Box-relative byte range | Size | Meaning |
| --- | ---: | --- |
| `0..<4` | 4 | Unsigned 32-bit big-endian total box size, `61464` |
| `4..<8` | 4 | ASCII type `uuid` |
| `8..<24` | 16 | TAP user type above |
| `24..<61464` | 61,440 | Fixed proof-slot payload |

When no slot exists, the in-memory photo producer appends this top-level box.
The locator parses top-level BMFF boxes, including ordinary 32-bit size,
large-size (`size32 == 1`), and to-end (`size32 == 0`) forms, and identifies the
slot by `uuid` plus the exact user type. A verifier MUST find exactly one match,
MUST require its payload length to be 61,440 bytes, and MUST use the located full
box offset/length in `assetHash.excludedRanges` and `proofSlot`.

Unknown boxes do not identify a slot. Zero matches are missing; multiple matches
are invalid. A verifier MUST NOT repair a duplicate or malformed input by
appending or choosing another slot.

## JPEG slot

The JPEG slot is an APP11 segment. The producer inserts it immediately after the
two-byte Start Of Image marker `FF D8`:

| Segment-relative byte range | Size | Meaning |
| --- | ---: | --- |
| `0..<2` | 2 | APP11 marker `FF EB` |
| `2..<4` | 2 | Unsigned 16-bit big-endian JPEG segment length, `61442`; JPEG length includes these two length bytes but not the marker |
| `4..<61444` | 61,440 | Fixed proof-slot payload |

The complete excluded segment length is therefore 61,444 bytes. The locator
walks JPEG marker segments after SOI until Start Of Scan (`FF DA`) or End Of
Image (`FF D9`) and identifies APP11 only when its declared length is 61,442 and
its payload begins with the exact TAP magic. This permits discovery even if a
metadata tool reorders pre-scan segments, while the producer placement remains
immediately after SOI.

A verifier MUST find exactly one match and use the full marker-through-payload
range for `assetHash.excludedRanges` and `proofSlot`. It MUST reject a malformed
segment, zero matches, or multiple matches rather than treating an arbitrary
APP11 segment as TAP evidence.

## Manifest, slot, and hash relationship

```text
primary HEIC/JPEG bytes
  contains XMP manifest JSON
  contains exactly one fixed proof-slot container

canonical(manifest.payload) -> metadataHash
photo bytes excluding full slot container -> assetHash
located slot offsets -> proofSlot descriptor
proof envelope in slot -> contentDigest + signingBinding + assertionObject
```

XMP is not excluded from the photo hash: it is covered once as part of the
format-native photo bytes and its payload is additionally named by
`metadataHash`. The proof slot is excluded so its envelope can be written after
hashing without changing `assetHash`.

## Container validation

A conforming verifier MUST:

- accept only the reviewed HEIC/HEIF image type as `heic` or JPEG as `jpeg` for
  these families, and match `capture.requestedCodec` (`hvc1` or `jpeg`);
- locate exactly one XMP manifest and exactly one proof slot;
- validate slot payload size, magic, version, positive signed-envelope length,
  capacity, and zero trailing padding;
- preserve exact file bytes and offsets when hashing;
- check auxiliary depth/disparity presence against both manifest availability
  fields and the binding's `depthResource`; and
- for Live Photo, treat `paired-video.mov` as a separate full-file signed
  resource rather than a nested photo-container region.

## Extraction notes

- The EXIF pointer contains the historical name `TAPDepthHEIC/1` even in JPEG
  output. This document preserves that current wire string and does not rename
  it; changing it requires a separate approved format task.
- The HEIC schema source comment predates first-class JPEG output. The container
  implementation and Product Contract make HEIC and JPEG separate reviewed
  choices, with the same XMP property and payload but different proof-slot
  containers.
- Reserved payload bytes `20..<24` are zeroed by producer construction but are
  not checked by current Swift or JS readers. This extraction records both facts
  and does not introduce a stricter reader rule under the existing v1 family.
