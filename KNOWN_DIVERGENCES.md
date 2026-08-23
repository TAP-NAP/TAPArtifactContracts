# Known Extraction Divergences

TAP-0094 is an authority extraction, not a format redesign. The following
differences were found while comparing current contracts and implementations.
They are recorded here so documentation does not silently choose new behavior.

## Photo `proofs` source comment

One Swift schema comment says proof data belongs in the manifest `proofs`
array. Current producer validation, verifier behavior, and specialized contracts
instead require the array to be present and empty; the proof envelope lives in
the fixed proof slot. This repository documents the latter current wire rule.
The stale source comment remains an implementation-repository cleanup item.

## HEIC-named EXIF pointer on JPEG

The non-authoritative EXIF `UserComment` literal is
`TAPDepthHEIC/1; metadata=xmp:tapdepth:Manifest`, including on current JPEG
output. The authoritative manifest is XMP `tapdepth:Manifest`. This repository
records the existing literal without renaming it; a rename would be a separate
format decision.

## Missing photo availability fields

The current Swift decoder defaults missing `capture.depthAvailability` and
`depth.availability` values to `available`, while current producers write both
fields and public-format routing is otherwise fail closed. The manifest field
contract therefore describes both fields as required for conforming new v1
artifacts. Whether the decoder fallback should be removed is a separate runtime
task.

## TAP Video semantic validation breadth

The producer contract defines the complete TAP Video payload groups. The
current TypeScript verifier parser performs strict family/proof routing but only
semantically validates a subset of those fields before retaining the rest as
untyped data. This repository documents the complete producer contract; it does
not claim the current verifier already enforces every field-level rule.

## TAP Video `mebx` local-key mapping

The QuickTime timed-metadata sample wrapper carries a track-local key ID that
must resolve through the sample description's metadata key table to
`mdta/com.tapnap.depth.klv`. The current TypeScript verifier observed during
TAP-0095 validates the atom size and only requires this ID to be non-zero before
reading the KLV bytes; its synthetic test happens to use `1`. It does not yet
validate the key-table mapping or the other reserved ID. The container contract
records the producer/QuickTime relationship; stricter runtime enforcement is a
separate verifier change.

## No-depth photo binding reconstruction

The producer emits an explicit unavailable `depthResource` when a Still or Live
Photo contains no auxiliary depth: `presence: unavailable`, `binding:
not-present`, `interpretation: no-depth-captured`, and
`platformPresenceCheck: AVDepthData-readback-missing`. The current Rust verifier
reconstruction observed during TAP-0094 still constructs only the available /
required form. Complete content-digest comparison therefore needs a verifier
follow-up before no-depth photos can be claimed cross-implementation conformant.
This documentation records the producer/Product Contract behavior and does not
change verifier code under the documentation-only task.

## Composite TAP Video fixture manifest drift

TAPCamDemo's repository-local
`Docs/Fixtures/TAPVideoManifestV1GoldenVectors.json` is still loaded by a Swift
decoder/KLV test. Its exact `depthFrame` bytes remain useful and are extracted
as this repository's KLV/zstd golden vector. Its separate manifest object uses
older optional/null and synchronization/software example values, so that object
is not a normative current-v1 example. The current manifest contract and
synthetic manifest example in this repository remain authoritative. Removing or
rewriting the local composite fixture requires its executable test dependency
to be updated separately; TAP-0095 does not promote the drift.

## Canonical JSON edge cases

Current Swift producers use `JSONEncoder` with `.sortedKeys` and
`.withoutEscapingSlashes`. Rust photo verification preserves the embedded
payload bytes for hashing, while TypeScript video verification has a separate
sorted-object encoder. The contract records current observable rules and
examples but does not rename them as RFC 8785. Additional byte examples are
needed before making broader claims about every floating-point or Unicode edge
case.

## Verifier extraction baseline

The Verifier working tree contained pre-existing uncommitted changes during the
initial extraction. Those files supplied current consumer evidence but are not
represented as a clean contract-repository release baseline. See
[SOURCE_SNAPSHOT.md](SOURCE_SNAPSHOT.md).
