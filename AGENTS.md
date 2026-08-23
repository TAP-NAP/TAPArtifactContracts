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

For the TAP-0094 bootstrap, resolve disagreements in this order:

1. [TAPCamDemo Product Contract](https://github.com/TAP-NAP/TAPCamDemo/blob/main/Docs/ProductContract.md).
2. [Approved TAP-0094 scope](https://github.com/TAP-NAP/TAPCamDemo/blob/main/Docs/ProjectBoard.md#tap-0094--establish-a-documentation-only-artifact-contract-repository).
3. Current specialized format and security contracts.
4. Current producer and verifier implementations and tests.
5. Historical documents and Git history.

After TAP-0094 completes and source repositories link to a released revision,
this repository owns the shared wire-level conventions. Product behavior,
runtime orchestration, UI, backend operation, and repository workflow remain in
their respective owner repositories.

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

1. Name the owning task and source snapshot.
2. Check identifiers and field names against producer and verifier evidence.
3. Parse every JSON example.
4. Scan local Markdown links and duplicate normative statements.
5. Record unresolved producer/verifier differences instead of silently choosing
   a new behavior.
6. Do not commit or push unless the product owner explicitly requests it.
