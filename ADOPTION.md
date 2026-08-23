# Producer and Verifier Adoption

This repository is adopted by reference, not as a software dependency.

## TAPCam mobile producer

TAPCamDemo owns capture behavior and the Swift types that serialize manifests,
bindings, and proof envelopes. Its Product Contract decides which artifact
families are current product capabilities. Repository-local module documents
describe packaging, pending signing, Photos export, and local-integrity flow.

Those documents refer here for shared field names, identifiers, byte-layout,
canonicalization, hash, resource-role, and compatibility conventions.

## TAP artifact verifier

Verifier repositories own their parsers, bounded-input handling, local content-
binding recomputation, report model, presentation, and backend calls. They refer
here for the same shared conventions and MUST report implementation gaps rather
than weakening this contract to match a permissive parser.

## No runtime coupling

Neither producer nor verifier imports this repository at runtime. Adoption does
not require a submodule, package-manager dependency, generated binding, schema
loader, or network request. A source repository records the reviewed contract
release or revision in its documentation and maintains its own implementation
and tests.

## TAP-0094 bootstrap status

- TAPCamDemo authority statements are updated in the current TAP-0094 working
  tree.
- TAPCamVerifier authority statements are updated by narrow additions to its
  current README and verification-flow document. The repository had pre-existing
  uncommitted documentation and runtime changes; TAP-0094 preserves them and
  does not claim or absorb them into this documentation extraction.
- No remote repository, tag, commit, or push is claimed by the initial local
  bootstrap unless separately recorded after owner approval.
