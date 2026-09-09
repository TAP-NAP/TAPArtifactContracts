# TAPCam Documentation Rules

Start with README and the relevant contract or design section. Each document
must define its subject without requiring application source or private records.

- Work from the requested scope, current source, and relevant tests. Clarify
  unresolved behavior without reopening established decisions.
- Write documentation for a new reader: purpose, concepts, usage, and limits.
  Keep private planning records, internal identifiers, and execution diaries out
  of source and documentation. Keep exact technical revisions where needed.
- Prefer existing seams and deletion of redundant material. Add no dependencies,
  generated infrastructure, or extra documentation without a current need.
- Keep changes and validation proportional; report actual checks and limits.
  Do not commit or push unless the current conversation authorizes it.

- Keep product requirements, manifest/wire contracts, design rationale, and
  capability-specific validation here. Requirements describe intended behavior;
  implementation and measured results require source/tests and actual evidence.
- Valid zero-depth captures are retained, signed, and exported. Consumer depth
  assessment remains separate from artifact integrity.
- Preserve exact identifiers, canonicalization, hash inputs, proof slots,
  resource sets, and fail-closed decisions. Follow the versioning policy for
  semantic/wire changes; editorial changes do not advance a consumer pin.
- Define fields, types, units, coordinates, and omission/null behavior in their
  owning contract. Keep examples synthetic and routing sidecars unsigned.
- Keep media, credentials, assertions, precise locations, application code,
  parsers, SDKs, and runtime dependencies out of this repository.
- Validate local links/anchors and JSON; check exact vectors when affected.
