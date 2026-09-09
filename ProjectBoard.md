# TAPCam Work Notes

Optional continuity notes, not a mandatory Kanban workflow. Read only an item
named by the user or directly needed for the current request. A bounded user
request authorizes a fix without a new Task, Board Steward, complete backlog
scan, or repeated confirmation of an established decision.

Add or update a short note only when unfinished work needs later continuation.
Do not keep owner/session boilerplate, duplicate queues, per-task Markdown, or
routine completed-fix records. Expired plans are inactive, not automatically
reopened; Done records are not proof of unexecuted device acceptance.

The [product contract](ProductContract.md) owns decisions and requirements.
[Acceptance.md](Acceptance.md) owns capability-specific procedures and their
actual evidence. Next unused legacy ID, only if needed: `TAP-0105`.

## Selected implementation gaps

These are approved but unfinished scopes (Todo), not instructions to start them.

<a id="tap-0010"></a>

### TAP-0010 — First-install App Attest (P0)

Replace Network-row `/healthz` completion with explicit initial App Attest
registration/verification, bounded attempts/timeout and manual Retry. Foreground
or status refresh must not restart it. Continue writes canonical credential-bound
S only after required setup facts; retained/restored receipts validate that
binding without trusting Keychain residue. Post-setup camera entry and local
capture stay network-independent. ProductContract §2.3 and §2.6–2.7 define the
settled behavior; this is implementation work, not a request for another decision.

Completion requires runtime and focused tests for explicit ownership, timeout,
manual Retry, canonical S, restore binding, and offline post-setup entry. Initial
bootstrap is separate from post-setup TAP-0015 and capture-proof verification.
Native evidence remains in Acceptance's first-install and production sections.

### TAP-0015 — Post-setup credential and Pending guards (P1)

Place W11 credential health/recovery and W12 credential/network-dependent Pending
recovery after committed t5, with camera-idle, protected-data, and credential
guards. Separate credential/assertion/export retry stages with bounded cooldown,
window/pause, one current persistence shape, and cancellation/stale handling.
Neither work may block route, preview, interaction, or local capture.

Complete when those placements, guards, stage retry, and regression tests agree
with ProductContract §2.4, §2.7 and §5–6 and the prototype's actual/target evidence.
Initial bootstrap/S/restore belongs to TAP-0010; delivered non-Network local
poster/recent-cover/App Intent release is not reopened.

### TAP-0090 — Non-Network lifecycle regression evidence (P0)

Validate delivered startup behavior without changing it or reopening TAP-0008/0009.
Use one public-safe versioned JSON/JSONL trace: schema, run/scenario, monotonic
sequence/clock, commit/configuration/platform/activation, S/P/I disposition,
route/event/workload, logical owner/executor, result and predecessor/correlation.
No media IDs, paths, URLs, hashes, keys, or credential identifiers.

Cover explicit permission/Camera/PhotoKit trigger boundaries; denied/restricted
recovery; targeted Camera/Photos-only permission recovery; observer-inert startup
and one eligible activation; independent I invalidation/atomic commit/interruption;
camera-plus-first-usable-catalog readiness; stable initialization with no failure
branch; and local poster/recent-cover/App Intent release after the committed
barrier. Map assertions once to stable prototype lifecycle/workload record IDs.

Complete when positive and prohibited-order cases actually run and emit parseable
trace/assertion artifacts consumable by the first-install/readiness procedures.
Keep structured logs and textual owner verdicts, not image/recording proof.
This excludes initial credentials (TAP-0010), W11/W12 (TAP-0015), real first-frame
performance, visual parity, and attended device verdicts. Failures belong to the
specific exercised boundary; they do not automatically broaden this work.

## Other approved follow-ups

All rows are Todo. The result column is the bounded completion condition; use
ProductContract for current decisions and clarify only an unsettled detail when
the owner selects the work.

| ID | Priority | Bounded result |
| --- | --- | --- |
| TAP-0012 | P0 | Code/output facts/UI agree on each Standard FOV's actual paired RGB/depth plan and fixed uncropped PRO; no new source switching, PRO zoom, or fusion. Device procedure: FOV/depth. |
| TAP-0013 | P0 | Dedicated-branch public-API Locked Camera experiment: launch/frame/capture/suspend/exit/relaunch, configuration committed before startRunning, containing-app import through app-level sessionContentUpdates, ready for attended acceptance. No production promotion, old-POC substitution, extension App Attest/Photos/network, private API, direct-open-as-import proof, or suppressing the normal queue to hide faults. |
| TAP-0014 | P0 | Owned attested captures present accurate persisted credential/verifiability state without standalone or Share-time Verify; UI/docs agree. External media Verify remains separate. |
| TAP-0016 | P2 | Define Video 3D depth time mapping, rendering, interaction, failure and performance before enabling it; complete approved contract/prototype, implementation/tests and required acceptance. Static-photo 3D is not synchronized Video 3D. |
| TAP-0017 | P2 | Add white balance through existing capability/intent/runtime boundaries, with product/prototype/runtime validation. No adjustable physical aperture. |
| TAP-0018 | P2 | Prototype-approved true source switching recomputes active depth/manual capabilities and handles failure; no preview-only zoom masquerading as source selection or unproven fusion. |
| TAP-0019 | P2 | Reviewed RAW/ProRAW profile, resources, manifest, signing, export, reader and validation contract plus accepted end-to-end implementation; do not mutate HEIC/JPG into a fallback. |
| TAP-0020 | P2 | Explicit 24 MP delivery/provenance contract with profile, runtime, storage, signing, reading and device acceptance. Existing quality tuning is not 24 MP support. |
| TAP-0021 | P2 | Design checks for future C2PA resource/signing compatibility, linked from the affected output/provenance designs and used when those designs change; no current compliance, certification or authority claim. |
| TAP-0023 | P2 | Real TAP Video package format, generation lifecycle, validation, UI and tests; do not disguise original MP4 as the Still/Live package. |
| TAP-0024 | P3 | Define Sticker supported media, rendering, privacy and lifecycle; complete approved contract/UI/implementation/validation, not placeholder activation. |
| TAP-0025 | P3 | Define Link ownership, privacy, expiry, backend and UX; complete approved implementation and operational acceptance, not a nonfunctional URL row. |
| TAP-0026 | P2 | Cancel superseded underlying PhotoKit carousel/display requests with explicit request ownership, cancellation/race tests; no Viewer navigation redesign. |
| TAP-0027 | P2 | Thumbnail retention, invalidation, purge triggers and protected-storage policy, implementation and tests (migration only if required); preserve user-visible thumbnails. |
| TAP-0029 | P3 | Move Debug fixtures to a support boundary and split oversized tests with approved structural boundaries and existing behavior tests passing; no product change or new-coverage claim from file moves. |

## Optional ideas

All rows are Inbox, not implementation authority. No native UI work begins from
an unapproved idea. Historical task dependencies do not force expired plans to
restart; select the current scope and verify its real prerequisites when needed.

| ID | Priority | Decision still needed / boundary |
| --- | --- | --- |
| TAP-0005 | P1 | Replace deprecated 照片保真 with owner-approved credential-readiness copy; synchronize locales/prototype without expanding authenticity claims. |
| TAP-0007 | P2 | Whether to resume 6.9-inch iPhone Store media, with actual media/rights; no iPad or treating prototype exports as app evidence. |
| TAP-0022 | P3 | Whether to accept external media and add true in-app Verify; separate from owned-capture local integrity. |
| TAP-0030 | P3 | Whether richer App Intents/widgets/controls/history are needed beyond the completed minimal surface. |
| TAP-0031 | P2 | Whether and where calibrated scores belong in Release UI. |
| TAP-0032 | P3 | Whether to propose Liquid Glass through the visual approval workflow. |
| TAP-0033 | P3 | Whether durable Library filters/reopen behavior are a product priority. |
| TAP-0034 | P3 | Replace SceneKit only if real performance/capability evidence requires it. |
| TAP-0035 | P3 | Whether a second shutter belongs in the product. |
| TAP-0036 | P3 | Whether handedness-based shutter placement belongs in the product. |
| TAP-0037 | P3 | Whether landscape side controls belong in the product. |
| TAP-0038 | P3 | Whether advanced per-frame video-depth analysis is in scope. |
| TAP-0039 | P3 | Whether more guide types have a current use. |
| TAP-0084 | P1 | Approve Photo/Live/Video grid↔Viewer zoom with stable chrome, visible/offscreen source, unready poster, cancellation, Reduce Motion and dismissal policy; preserve item identity and internal paging/Share/Delete/playback. Prototype and native parity precede acceptance. |
| TAP-0088 | P1 | After the startup/cold-path prerequisites actually hold and the owner selects it, simulate Viewfinder control/workload states, owner/queue, timing, cancellation and stale work; existing capabilities only, deterministic fixtures/visual QA, no native camera delivery. |
| TAP-0089 | P1 | Same explicit selection and actual startup prerequisites: simulate Photo RAW/2D/3D analysis states, loading/failure/cancel/stale work; show unsupported modes as deferred, retain fixture/visual QA, no native or Video 3D claim. |
| TAP-0091 | P1 | Freeze the actual FOV/front-rear/PRO reproduction, then preserve out-of-preview chrome across success/recovery; no new camera-source feature or assumed cause. Review intentional visual changes and retain focused regression/native before-after evidence. |

## Acceptance references

These identify sections of one procedure document, not ten more work items to
read or reconcile before coding. Select them only for the relevant requested run.

| Legacy ID | Procedure |
| --- | --- |
| TAP-0040 | [First-install explicit actions](Acceptance.md#tap-0040) |
| TAP-0041 | [Camera readiness](Acceptance.md#tap-0041) |
| TAP-0042 | [Professional controls](Acceptance.md#tap-0042) |
| TAP-0043 | [FOV and depth](Acceptance.md#tap-0043) |
| TAP-0044 | [Live Photo](Acceptance.md#tap-0044) |
| TAP-0045 | [Video / zero depth / iCloud](Acceptance.md#tap-0045) |
| TAP-0046 | [Production App Attest](Acceptance.md#tap-0046) |
| TAP-0047 | [Library permissions and deletion](Acceptance.md#tap-0047) |
| TAP-0048 | [Web-to-native parity](Acceptance.md#tap-0048) |
| TAP-0049 | [Locked Camera experiment](Acceptance.md#tap-0049) |

## Expired plan

<a id="tap-0083"></a>

**TAP-0083 — Expired on 2026-09-09.** Its former module-README/separate-standard
instructions and perpetual Doing assignment are retired. This is not a claim
that performance passed. ProductContract §2.7 retains the execution constraints;
[Cold-path performance](Acceptance.md#cold-path-performance) retains the complete
2×2 physical timing matrix and regression conditions. Do not automatically
reactivate this plan or allocate a replacement task.

## Completed in this change

<a id="tap-0011"></a>

**TAP-0011 — Done, 2026-09-09.** Zero-depth export was already approved. Removed
the terminal-ingest and worker-start depth guards; new valid zero-depth video
proceeds through signing/export/readback with integrity checks intact. The three
focused Swift suites passed 100 test definitions / 102 executions, zero failed
or skipped; build-for-testing passed. The Video device procedure is separate.
Old terminal records are not migrated. No product decision remains to reconfirm.

## Closed identifiers

### Done

| ID | Title |
| --- | --- |
| `TAP-0001` | Establish the canonical contract, UI workflow, and Markdown board |
| `TAP-0002` | Reconcile active documents with the Product Contract |
| `TAP-0003` | Migrate unique facts, then remove obsolete documents |
| `TAP-0004` | Normalize TAP Library and Pending Capture Queue terminology |
| `TAP-0006` | Build the repository-owned HTML/Web UI prototype foundation |
| `TAP-0008` | Enforce explicit-action-only first-install operations |
| `TAP-0009` | Enforce first-frame camera-interactive readiness |
| `TAP-0050` | Required versus optional first-install rows |
| `TAP-0051` | Bounded Network retry policy foundation |
| `TAP-0052` | PRO Photo and TAP Video Release capability |
| `TAP-0053` | Standard Basic EV and PRO manual controls |
| `TAP-0054` | HEIC/JPG Release output formats |
| `TAP-0055` | No-depth still-photo non-blocking output |
| `TAP-0056` | Live Photo capture, signing, export, and playback chain |
| `TAP-0057` | TAP Video core chain and foreground RAW/2D playback |
| `TAP-0058` | Mixed-media user-facing TAP Library |
| `TAP-0059` | Photos-style Viewer and bottom capsule |
| `TAP-0060` | Static-photo native 3D point projection |
| `TAP-0061` | App-owned, on-demand Share flow |
| `TAP-0062` | Cross-project Live Photo browser verification contract |
| `TAP-0075` | Migrate and remove obsolete Startup/Camera design documents |
| `TAP-0076` | Migrate and remove obsolete Viewer/Queue design documents |
| `TAP-0077` | Consolidate App Attest documentation |
| `TAP-0078` | Extract the canonical TAP Video format contract and remove old video plans |
| `TAP-0079` | Migrate historical evidence and remove AITrace, dated scorecards, and old acceptance snapshots |
| `TAP-0080` | Migrate Locked Camera experiment guardrails and remove main-tree experiment prose |
| `TAP-0081` | Unify TAP Share selection and preparation into an anchored handoff |
| `TAP-0082` | Device acceptance: TAP Share anchored handoff and anti-flash progress |
| `TAP-0085` | Restrict the current runtime target to iPhone |
| `TAP-0086` | Remove Locked Camera integration and entry points from main |
| `TAP-0087` | Define startup lifecycle, heavy-work budgets, and an instrumented prototype |
| `TAP-0092` | Clean up brittle tests, unmounted legacy code, and redundant repository information |
| `TAP-0093` | Freeze pre-release v1 schemas and remove developer compatibility layers |
| `TAP-0094` | Establish the documentation-only TAPArtifactContracts repository |
| `TAP-0095` | Deduplicate migrated artifact-contract prose across source repositories |
| `TAP-0096` | Extract the HTML/Web prototype into TAPCamPrototype |
| `TAP-0097` | Commit the prototype extraction in both local repositories |
| `TAP-0098` | Publish TAPCamDemo and create the private TAPCamPrototype remote |
| `TAP-0099` | Move the canonical Kanban into TAPCamKanban and remove the Demo-owned copy |
| `TAP-0100` | Close TAPCamDemo producer conformance gaps against the reviewed artifact contracts |
| `TAP-0101` | Complete TAPCamVerifier artifact-contract validation |
| `TAP-0102` | Align and simplify TAPCamVerifier while preserving the easter egg |
| `TAP-0103` | Clean TAPArtifactContracts repository and revision terminology |
| `TAP-0104` | Collapse redundant documentation to one authority per responsibility |

### Deprecated

| ID | Title |
| --- | --- |
| `TAP-0028` | Migrate legacy exported GPS copies |
| `TAP-0063` | First-install page appearance implicitly requests permissions |
| `TAP-0064` | Video and Live Photo are globally unimplemented |
| `TAP-0065` | PRO is Photo-only |
| `TAP-0066` | Standard FOV is preview-only magnification |
| `TAP-0067` | Tool drawer, vertical Verify/dismiss, detents, and top-level analysis modes |
| `TAP-0068` | Direct resource preparation followed immediately by system Share |
| `TAP-0069` | Verify every TAPCam-owned capture during view or Share |
| `TAP-0070` | Treat the existing Locked Camera POC as a product capability |
| `TAP-0071` | Discard still photos or TAP Video only because depth is missing |
| `TAP-0072` | TAP Video AirPlay, PiP, and background playback |
| `TAP-0073` | Per-frame depth in the current Live Photo MOV contract |
| `TAP-0074` | Simplified Chinese `Photo Integrity = 照片保真` |

## Historical lookup

The pre-simplification detailed records are preserved by
`TAPCamKanban@3f5873d2a85fcdda4577499d67fe3aeac6913536:ProjectBoard.md`;
earlier records use
`TAPCamDemo@8171e00fc37b4281952553456b422e6eef43c3e1:Docs/ProjectBoard.md`.
Read history only when a historical decision is actually needed.

TAP-0104's former detailed completion record was not found in the visible Git
history; do not reconstruct an owner verdict. Its recorded cleanup commits are
Demo `955cfdfe`, Verifier `b4871657`, Prototype `f249c7d0`, Contracts `21e5011c`,
and Kanban `3f5873d2`. This provenance gap is historical context, not a standing
cleanup task or a reason to block unrelated work.
