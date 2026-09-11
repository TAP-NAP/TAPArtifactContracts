# Photo Containers v1

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

Producers write the complete manifest object with `schema`, `payload`, and empty
`proofs`. A verifier uses only the fixed slot for proof. It MUST resolve the XMP property, undo XML
entity encoding, parse the resulting JSON, and require exactly one manifest
property. XML attribute-style and element-style representations of the same
namespace/property are container serializations, not different manifest
families.

For HEIC, the property is XMP metadata associated with the primary image. Its
physical HEIF metadata-item offsets are not fixed by this contract; discovery is
by the XMP namespace and property, not a hard-coded box offset. For JPEG, the
property is in the JPEG XMP metadata carried by APP1; the APP1 byte location may
vary after metadata-preserving ImageIO output.

The producer starts from the format-native captured file, injects XMP through a
metadata-preserving copy rather than decoding and re-encoding pixels, and
requires exact JSON-string readback before reserving the proof slot. A verifier
uses the resulting bytes and MUST NOT reconstruct the signed artifact from
decoded image pixels.

The producer also writes this exact EXIF UserComment pointer in both reviewed
containers:

```text
TAPDepthHEIC/1; metadata=xmp:tapdepth:Manifest
```

It is a discovery hint only. It is not the manifest, a family identifier, a
content hash, or proof evidence.

Both wrappers carry the common
[61,440-byte proof-slot payload](../bindings/capture-binding-and-proof-v1.md#proofslot).
This document defines only the enclosing photo-container ranges.

## HEIC/BMFF slot

The HEIC slot is one top-level ISO BMFF `uuid` box with 16-byte user type:

```text
hex:   54 41 50 43 41 4d 50 52 4f 4f 46 53 4c 4f 54 31
ASCII: TAPCAMPROOFSLOT1
```

The v1 producer layout for a normal 32-bit-size box is:

| Box-relative byte range | Size | Meaning |
| --- | ---: | --- |
| `0..<4` | 4 | Unsigned 32-bit big-endian total box size, `61464` |
| `4..<8` | 4 | ASCII type `uuid` |
| `8..<24` | 16 | TAP user type above |
| `24..<61464` | 61,440 | Fixed proof-slot payload |

When no slot exists, the producer appends this top-level box before signing.
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
