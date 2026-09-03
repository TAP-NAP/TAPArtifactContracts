# TAP Artifact Contracts Agent Rules

This repository is the documentation-only authority for the artifact contracts
shared by TAPCam producers and TAP artifact verifiers.

## Repository boundary

- Store normative prose, field tables, identifiers, binary-layout diagrams, and
  synthetic JSON examples only.
- Do not add application code, parsers, validators, SDKs, generated bindings,
  code generators, package-manager manifests, or runtime dependencies.
- A manifest instance produced for one capture remains embedded in that capture
  artifact. This repository documents its shape; it does not store user media or
  replace the embedded instance.
- Do not add real captures, App Attest credentials, assertion objects, device
  identifiers, precise locations, or other user data. Examples must be synthetic.

## Authority and change order

Task IDs and delivery status are owned by the sibling
[TAPCamKanban Project Board](../TAPCamKanban/ProjectBoard.md). Read the complete
matching Task before changing this repository. The Board may authorize work but
is not required to interpret a released artifact contract.

This repository must be interpretable without a producer or verifier checkout.
Resolve contract questions in this order:

1. The current [contract index](CONTRACTS.md) and
   [versioning policy](VERSIONING.md).
2. The normative document that owns the affected family or container.
3. The synthetic examples and exact vectors indexed by
   [examples/README.md](examples/README.md).
4. This repository's Git history for historical context only.

An approved product decision may authorize a contract change, but the complete
resulting rule must be recorded here. Do not require a private task, source
file, checkout, or implementation link to interpret the released contract.

This repository owns the shared wire-level conventions. Product behavior,
runtime orchestration, UI, backend operation, and repository workflow remain in
their respective owner repositories.

Do not store producer/verifier source snapshots, adoption ledgers, executable-
mirror inventories, or implementation-gap trackers here. Downstream repositories
own those local records and link to this repository; this repository does not
link back to them.

## Normative writing rules

- Use `MUST`, `MUST NOT`, `SHOULD`, and `MAY` only for normative requirements.
- Define every field's JSON name, type, required/optional/null/omitted behavior,
  unit, coordinate system, enumeration, and consumer obligation.
- Keep Still Photo, Live Photo, TAP Video, content-binding, proof, registration,
  and transport schema families distinct even when they all use version `v1`.
- Preserve current serialized names, identifiers, canonicalization rules, hash
  inputs, proof semantics, and embedding locations. A behavioral or wire-format
  change requires a separately approved task and a new compatible or breaking
  family revision as appropriate.
- Treat `tapcam-export.json` as unsigned routing metadata, never as capture
  authenticity evidence.
- Keep repository-relative links valid. Every JSON example must parse with a
  standard JSON parser and state whether it is accepted or rejected.

## Review checklist

Before handing off a contract change:

1. Name the affected contract and change classification.
2. Check identifiers and field names against the owning normative document and
   examples.
3. Parse every JSON example.
4. Scan local Markdown links and duplicate normative statements.
5. Keep implementation mismatches in the affected downstream repository instead
   of importing its conformance ledger here or silently changing this contract.
6. Do not commit or push unless the product owner explicitly requests it.
