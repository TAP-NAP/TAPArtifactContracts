# TAPCam Documentation Rules

This is the main documentation repository. Start with README and read only the
contract or design section needed for the current request. Normative documents
must be understandable here without another repository or a task record.

## Keep work small

- The user's explicit request authorizes a bounded fix or documentation cleanup.
  Do not require a Task ID, Board Steward, full backlog scan, or repeated
  confirmation of a settled decision. Source inspection is required only when
  making or checking an implementation claim.
- ProjectBoard is optional work notes for continuity. Read a matching item only
  when relevant. Add a compact record only when work needs later continuation;
  do not create records for routine completed fixes. Expired plans are inactive,
  not a standing instruction to reopen work.
- Keep product requirements, manifest/wire contracts, design rationale, and
  acceptance procedures here. Consolidate related procedures into sections;
  create no per-task Markdown, duplicate indexes, status copies, or audit diaries.
- Product requirements are normative intent, not a claim that every behavior
  ships. Tests/builds and dated acceptance establish implementation and evidence.
  Keep incomplete implementation notes in ProjectBoard, separate from policy.
- Zero-depth still photos and TAP Video are retained, signed, and exported.
  Record depth availability honestly; downstream depth assessment is separate
  from capture integrity. This is an established decision, not an open proposal.

## Preserve contracts

- The contract index and versioning policy govern shared formats. Preserve exact
  identifiers, bytes, canonicalization, hash inputs, proof slots, resource sets,
  and fail-closed decisions. Semantic/wire changes require owner direction and
  the appropriate family revision; an editorial move does not change a pin.
- Define fields, types, units, omission/null behavior, coordinates, and consumer
  obligations in one owning document. Examples remain synthetic; routing
  sidecars are unsigned and are not authenticity evidence.
- Keep actual media, credentials, assertions, precise locations, parsers, SDKs,
  application code, generated bindings, and runtime dependencies out of this repo.
- Local README/AGENTS in implementation repositories own their run commands and
  implementation map. Evidence may name a source revision or artifact, but may
  not substitute for a missing normative definition here.
- Validate local links/anchors and JSON; check exact vectors when affected.
  Report real evidence limits. Do not commit or push unless requested.
