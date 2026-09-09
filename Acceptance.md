# Device and Visual Acceptance

Read only the capability section relevant to the requested run. These are
executable procedures, not ten separate tasks or a mandatory coding checklist.
A document move does not execute a procedure or change its recorded verdict.

Before a device run, freeze the build/commit, device/iOS, reset method, selected
matrix, run budget, evidence method, and pass/fail/blocked conditions with the
owner. Existing approval for the same scope remains valid; do not ask again.
No destructive reset, production operation, or unselected matrix is implied by
an ordinary code change. Record actual results by step and retain the owner
verdict; a build or simulated fixture is not physical-device acceptance.

All sections below remain unexecuted in full unless their own evidence says
otherwise. The bounded 2026-08-15 startup observation and separately accepted
Share run do not close the remaining matrices. Record the actual build and
operator when a section is exercised, not placeholder metadata in every draft.

- [First-install actions](#tap-0040)
- [Camera readiness](#tap-0041)
- [Professional controls](#tap-0042)
- [FOV and depth alignment](#tap-0043)
- [Live Photo](#tap-0044)
- [TAP Video, including zero depth](#tap-0045)
- [Production App Attest](#tap-0046)
- [Library permissions and deletion](#tap-0047)
- [Web-to-native parity](#tap-0048)
- [Locked Camera experiment](#tap-0049)
- [Cold-path performance](#cold-path-performance)

<a id="tap-0040"></a>

## First-Install Explicit Operations (TAP-0040)
- Status: `Draft — not executed`
- Related Delivery: `TAP-0008`, `TAP-0010`, `TAP-0090`
- Contract: `ProductContract §2.1–2.3`

This is an attended physical-device procedure. Before execution, the Agent must
present the filled build, device, reset method, logging surface, and full scope
to the product owner. Simulator permission fixtures are not a substitute.

The 2026-08-15 `fix0809` native candidate implements the non-Network
permission/recovery and observer boundaries in this procedure. It deliberately
does not migrate `/healthz` and initial App Attest behavior; that work remains
owned by `TAP-0010`. This record therefore remains Draft and cannot receive a
Pass verdict from that candidate alone.

The branch-executable subset is limited to Camera/Photos row ownership,
denied/restricted recovery, Required Permission Check targeted refresh, and the
PhotoKit observer/catalog activation boundary. Steps that require initial App
Attest registration, `/healthz` replacement, or retry delivery remain blocked;
their current behavior must not be counted as target acceptance. `TAP-0090`
owns the non-Network structured event-order regression assertions used by this
attended procedure.

### Preconditions

- The candidate build contains the explicit-action and bounded Network retry
  fixes and can log, with public-safe timestamps: setup appearance, button tap,
  each permission request, each first-install App Attest
  challenge/registration/verification attempt and timeout, PhotoKit
  observer/fetch/backfill/write, and camera warmup. Logs do not expose
  credential material or key identifiers.
- The device can return Camera, Photos, Location, and Microphone to
  `.notDetermined` and can be taken offline.
- Structured device-log capture and its automated event-order assertions are
  ready before process launch. The retained evidence contains no photo,
  screenshot, or screen recording.
- The `TAP-0090` public-safe versioned JSON/JSONL trace schema and assertion
  report are available for the non-Network events consumed by this procedure.
- **Owner live:** approve a Delete-and-Reinstall of the test app. If that reset
  does not restore `.notDetermined`, separately approve any system-level
  privacy reset.

### Reset And Install

1. Archive earlier logs; do not delete existing Photos media.
2. Perform Delete-and-Reinstall with the pinned signed build.
3. Confirm all four OS permission states are `.notDetermined`; otherwise stop
   as Blocked.
4. Begin the structured device log before the first Foreground Process Launch.
5. Run one offline Delete-and-Reinstall round and one separately reset
   Delete-and-Reinstall optional-skip round.

### Procedure And Expected Results

1. **Owner live:** launch and leave the page untouched for at least 10 seconds.
   No system prompt, Network attempt, observer, protected fetch/write, or camera
   warmup may occur.
2. Background and foreground the app without tapping. Only passive status
   refresh is allowed.
3. With the device offline, tap the Network action once. One bounded sequence
   may attempt initial App Attest work with policy-timed automatic retries and
   must end at its total timeout. A generic `/healthz` success is not the row's
   completion condition.
4. After timeout, background/foreground, wait at least one retry interval, then
   restore the network without tapping Retry. Displayed state may refresh, but
   no new Network sequence may start.
5. **Owner live:** tap Network Retry. Exactly one new bounded sequence starts;
   the row succeeds only after initial App Attest registration/verification is
   complete.
6. **Owner live:** tap Camera, Photos, Location, and Microphone Allow one at a
   time, resolving each system prompt before the next tap. Each tap must map to
   exactly its own request and must not start another row.
7. Perform Delete-and-Reinstall again. After the untouched and
   background/foreground checks,
   complete Network, Camera, and Photos; use Skip for Location and Microphone.
   Neither Skip may show a system prompt.
8. **Owner live:** confirm Continue is enabled only after App Attest, Camera,
   and Photos are complete and that the two optional rows remain skippable. Do
   not press Continue in this procedure; readiness belongs to `TAP-0041`.
9. In separate reset rounds, deny Camera, Photos, Location, and Microphone one
   row at a time. Confirm each denied row owns its **Open Settings** recovery;
   a restricted fixture shows stable restricted guidance and does not promise
   a Settings recovery that the system policy cannot provide.
10. After Setup has completed, revoke Camera and then Photos. On each launch or
    foreground return, confirm the root enters Required Permission Check with
    only Camera and Photos represented and with no Continue button.
11. Recover the affected required permission from its row/system boundary.
    Confirm the targeted return refresh does not start Network, Location, or
    Microphone work and automatically re-evaluates the route.
12. For Photos, correlate observer registration and catalog logs: no observer
    or catalog scan before the explicit eligible boundary; exactly one
    idempotent observer activation afterward; revocation/deactivation cancels
    queued catalog refresh work.

### Required Evidence

- One structured, monotonically ordered event log for each
  Delete-and-Reinstall round. It records the app-owned action, system-boundary
  entry/return, observed authorization enum, and resulting route without
  retaining the system permission sheet as an image or video.
- Automated assertions over that log proving the action/request/Network order
  and the absence of early PhotoKit work or camera warmup.
- The machine-readable `TAP-0090` assertion report, with each non-Network result
  mapped to its stable TAP-0087 lifecycle/workload record ID.
- Textual Camera/Photos/Location/Microphone authorization snapshots before each
  round, recorded as public-safe enum values rather than screenshots.
- A structured App Attest attempt table covering explicit start, bounded
  automatic retries, timeout, background/foreground, connectivity recovery,
  manual Retry, and final registration/verification success without private
  credential values.
- Pinned build/commit, device model, and iOS identifiers.
- **Owner live:** a retained textual verdict confirming that no surprise prompt
  appeared and every operation followed its corresponding action.

Do not save a photo, screenshot, or screen recording as acceptance proof. The
structured logs, automated assertions, and owner-live textual verdict are the
durable evidence for this procedure.

This procedure proves explicit trigger boundaries. Launch timing is checked
separately under [Cold-path performance](#cold-path-performance).

### Verdict

- **Pass:** every step completes, all trigger boundaries hold, and the owner
  accepts both first-install flows.
- **Fail:** any appearance/refresh/lifecycle/background path starts an operation;
  a row starts another row; post-timeout Network restarts without Retry; an
  optional Skip requests permission; Network is marked complete by reachability
  without App Attest completion; or evidence cannot prove hidden work stayed
  inactive.
- **Blocked:** permissions cannot be reset; network conditions cannot be
  controlled; logging is insufficient; or required deletion/privacy-reset
  approval is absent.

<a id="tap-0041"></a>

## First Camera-Interactive Readiness (TAP-0041)
- Status: `Draft — bounded fix0809 device path accepted; full matrix open`
- Related Delivery: `TAP-0009`, `TAP-0010`, `TAP-0015`, `TAP-0090`
- Contract: `ProductContract §2.4–2.7`
- Build/Commit: `TAPCamDemo@66ac0019e3768d91317c6726f772c5ea0d2d9fa2`
- Device/iOS: `Owner-attended physical device; exact identifiers not recorded in this evidence`
- Human Confirmation: `Accepted 2026-08-15 for the bounded fix0809 non-Network executed path; full procedure pending`

This attended procedure proves the ordering that prevents early camera entry
and an unresponsive startup. The Agent must first agree with the owner on a
safe method for delaying or failing readiness; an unreviewed fault injection is
not allowed.

The 2026-08-15 `fix0809` native candidate contains the independent marker,
camera-plus-catalog gate, stable Resource Initialization surface, and local
poster/recent-cover release needed for this procedure. The owner exercised and
accepted that bounded lifecycle path on a physical device. Post-Setup App
Attest and credential/network-dependent Pending scheduling were not migrated
and remain owned by `TAP-0015`, so the complete deferred-release target is not
implemented by this candidate.

For this branch, only the non-Network candidate subset is executable: validate
Camera/Photos → Initialization routing with an existing compatible Setup fact;
exercise the two readiness groups, marker interruption/invalidation, committed
Viewfinder-frame barrier, local poster/recent-cover release, and permission
recovery. Canonical credential-bound Setup receipt creation and the App Attest/
Pending children of deferred release remain blocked on `TAP-0010` and
`TAP-0015`. The bounded device acceptance recorded below does not assign this
full procedure a `Pass` verdict or substitute for its unexecuted scenario
matrix. `TAP-0090` owns the non-Network structured ordering assertions reused by
this attended procedure.

### 2026-08-15 Bounded Device Observation

The owner ran the current `fix0809` working tree on a physical device, supplied
the resulting lifecycle log, and accepted the experienced non-Network startup
flow. The log records the usable TAP Library catalog publication before the
camera readiness group, followed by initialization-marker commit and then the
local recent-cover refresh release. The owner also exercised the resulting
Viewfinder/Library experience without reporting an early marker, black or
stalled preview, or unsafe first interaction.

This observation covers only the executed path. It does not cover the second
interruption round, readiness fault injection, Camera/Photos revocation and
recovery, In-place App Update, restore/migration, exact marker-file inspection,
or instrumented physical timing. Canonical credential-bound `S`, Network, and
post-`t5` App Attest health plus Pending Capture recovery remain outside the
accepted candidate scope under `TAP-0010`/`TAP-0015`.

### Preconditions

- `TAP-0040` has passed for the same setup behavior, or the same boundary has
  been reconfirmed for this build.
- The build logs required rows complete, Continue, separate Setup-receipt write,
  capture graph/path ready, real first preview frame presented,
  controls/shutter safe, haptics prepared, first usable Library catalog
  metadata snapshot, initialization-marker write, Viewfinder entry, deferred
  App Attest health/recovery, and Pending Capture Queue retry.
- A reviewed readiness delay/failure method and marker inspection are available.
- The `TAP-0090` public-safe versioned JSON/JSONL trace schema and assertion
  report are available for the non-Network ordering consumed here.
- **Owner live:** the owner can observe the real frame, controls, shutter, and
  haptic response.

### Reset And Install

1. Perform Delete-and-Reinstall with the pinned build; confirm setup is
   incomplete.
2. Complete the Network row's first-install App Attest bootstrap; then authorize
   Camera and Photos; optionally skip Location/Microphone.
3. Start the structured device log before pressing Continue. Do not start or
   retain a photo, screenshot, or screen recording as proof.
4. Reserve a second Delete-and-Reinstall round for interruption/failure
   recovery.

### Procedure And Expected Results

1. **Owner live:** verify that required rows completing does not leave setup.
2. Tap Continue. Confirm the Setup receipt is written, then a Resource
   Initialization surface remains while both readiness groups progress:
   camera graph/path, real first preview frame, safe shutter/primary controls,
   and haptics; plus the first usable Library identity/order metadata snapshot.
3. Try shutter and primary controls during readiness. Inputs must be safely
   blocked and must not start capture or an unsafe state transition.
4. Record the exact event order. The versioned initialization marker and
   Viewfinder transition must occur only after every camera condition and the
   Library snapshot, not after session configuration or either group alone.
   Inspect the marker as one Application Support JSON value and verify its
   bundle, version, build, initialization schema, installation generation, and
   local device-generation fields.
5. **Owner live:** immediately after dismissal, capture once and operate a FOV
   plus a primary mode/control. Confirm a real frame, normal haptic, no UI stall,
   black screen, or unresponsive control.
6. On the second Delete-and-Reinstall round, use the approved method to terminate after
   Continue but before both readiness groups complete. Relaunch: the Setup
   receipt remains valid, the initialization marker remains absent/stale, and
   Resource Initialization returns without replaying Setup.
7. Delay each readiness group separately. Confirm the stable Resource
   Initialization page exposes no product Failed, Retry, timeout, skip, or
   degraded-entry action; bounded diagnostics identify the incomplete group.
8. Complete readiness, perform another Foreground Process Launch with a Cold
   Resource Path, and confirm both Setup and Resource Initialization are
   silently skipped when their independent facts are valid.
9. Revoke Photos, perform a Foreground Process Launch, restore it through
   Required Permission Check,
   then repeat for Camera. Neither change may replay Setup. The permission page
   must automatically re-evaluate to Resource Initialization when its marker is
   stale or Viewfinder when it is current.
10. Perform an owner-approved In-place App Update with the Setup receipt
    retained and the initialization marker stale. Start offline. **Owner live:**
    confirm Resource Initialization and first interactive camera entry do not
    wait for later App Attest health/recovery or Pending Capture Queue work. A
    Development Replacement Install is separate diagnostic evidence and does
    not substitute for this update scenario unless it is recorded explicitly.

### Required Evidence

- A structured Continue-to-first-interaction event timeline with monotonic
  ordering and automated assertions.
- Setup-receipt and initialization-marker timestamps relative to Continue,
  graph readiness, first real preview, safe controls/haptics, and Library
  catalog publication.
- Structured interruption/relaunch events plus a public-safe textual marker
  inspection containing only schema/runtime/install/device-generation status,
  never credential material.
- A capture-success event and control/haptic interaction events after readiness;
  no captured image is retained as proof.
- Foreground Process Launch logs after Photos and Camera revocation, including
  Required Permission Check and automatic post-recovery routing.
- Offline App Attest/queue scheduling logs showing camera readiness remains
  independent. A deliberately rejected Debug attestation is not a lifecycle
  failure verdict; only its scheduling position is relevant here.
- Automated assertions for the required ordering and prohibited-early-work
  conditions.
- The machine-readable `TAP-0090` assertion report, with each result mapped to
  its stable TAP-0087 lifecycle/workload record ID.
- **Owner live:** a retained textual verdict confirming the real preview,
  haptic, safe controls, and absence of a UI stall.

Do not save a photo, screenshot, or screen recording as acceptance proof. The
structured logs, automated assertions, and owner-live textual verdict are the
durable evidence for this procedure.

This procedure proves readiness ordering. Launch timing is checked separately
under [Cold-path performance](#cold-path-performance).

### Verdict

- **Pass:** both readiness groups precede marker/entry; interruption cannot
  leave a false marker; later Camera/Photos changes use Required Permission
  Check without replaying Setup; the owner accepts the first interaction.
- **Fail:** the marker is early; a fake, black, or stalled preview appears; shutter or
  controls are unsafe; background credential work blocks entry; or permission
  changes reopen setup.
- **Blocked:** first-frame/marker ordering is not observable; no approved
  readiness failure method exists; the device lacks camera/haptic capability;
  or the owner cannot attend.

<a id="tap-0042"></a>

## Professional Camera Control Matrix (TAP-0042)
- Status: `Draft — not executed`
- Related Delivery: `TAP-0053`
- Contract: `ProductContract §3.1`

The owner must freeze the device/iOS matrix before execution. At minimum it
should include one eligible LiDAR Pro device and one non-eligible device. A
single-device result must never be generalized to an untested family.

### Preconditions

- Fixed near/far focus targets, a high-contrast exposure target, controllable
  light, tripod, recording, and public-safe diagnostics are available.
- Readback includes active device/format, ISO, shutter, lens position, exposure
  target offset, AF/MF state, generation, clamp, and risk state.
- Camera preferences, Basic EV, ISO, shutter, and focus are reset.
- **Owner live:** judge exposure direction, control feel, focus plane, frosted
  transitions, and black-screen risk.

### Procedure

1. In Standard, confirm Basic EV and FOV selector work and Standard did not
   silently choose the LiDAR PRO source.
2. **Owner live:** enter PRO. Frost must remain until a real preview is
   interactive. Confirm rear LiDAR 24 mm / 1x, `EV / ISO / S / Focus / ƒ`, no
   Basic EV/FOV, and read-only `ƒ`.
3. In `A/A`, move EV positive and negative; compare perceived brightness with
   `exposureTargetOffset` direction.
4. Adjust ISO through middle and device limits (`M/A`), restore Auto, then do
   the same with shutter (`A/M`).
5. Set both manually (`M/M`): both values persist and EV becomes a read-only
   meter. Restore one parameter at a time and finally return to `A/A`.
6. **Owner live:** tap AF, drag the temporary EV rail, and confirm its focus
   anchor does not move. A new tap resets temporary EV. Long press locks AF/AE;
   ordinary focus events must not clear the locked overlay.
7. Confirm an unlocked focus overlay does not disappear by a fixed timer and
   clears on real runtime invalidation.
8. Open Focus without moving it: AF remains. The first real drag enters MF and
   locks the current lens position before numeric writes.
9. Drag MF continuously and verify latest-wins with no stale value jump.
10. **Owner live:** tap near/far targets in MF. Focus-only assist must not change
    exposure, and shutter stays disabled until honest locked readback. Rapid taps
    and slider movement must reject stale request results.
11. Check the MF loupe: tapped position is centered, the main preview stays 1x,
    and disabling the magnifier does not disable focus assist.
12. Switch PRO to front and back. Frost remains until interactive; front exposes
    no PRO/MF, and returning restores suspended rear PRO intent without
    rewriting the saved preference.
13. Switch PHOTO/TAP VIDEO and confirm the same eligible PRO device/control
    contract; recording stress belongs to `TAP-0045`.
14. Check Default Off, Default On, Remember Last State, and safe Standard
    fallback.
15. Repeat availability/startup/front/back checks on the non-eligible device;
    PRO must not appear and Standard remains usable.

### Expected Results

- EV direction, preview, and readback agree; ISO/S Auto and Manual modes clamp
  honestly and expose no impossible values.
- AF/MF/assist results belong to the active device and generation; assist never
  writes exposure or leaves UI/device mode disagreement.
- The main preview, loupe, and recorder do not create a second hardware preview
  or crop the primary view.
- Front camera never exposes MF; every Standard/PRO and rear/front transition
  keeps frost until the destination preview is interactive.

### Required Evidence

- Capability/readback logs, continuous video, EV/ISO/S/Meter/lens table, and
  AF/MF/generation timeline for each device.
- Eligible/non-eligible screenshots and any generated capture artifacts.
- **Owner live:** written acceptance of exposure, focus, loupe, transition feel,
  and absence of black screens.

### Verdict

- **Pass:** the frozen matrix completes with no stale write, false capability,
  black screen, or mode mismatch, and the owner accepts subjective behavior.
- **Fail:** direction/readback mismatch; unsupported controls appear; AF/MF
  results go stale; assist changes exposure; front MF appears; or frost leaves
  before a real destination preview.
- **Blocked:** the matrix, required devices, diagnostics, stable targets, or
  owner attendance are unavailable.

<a id="tap-0043"></a>

## Standard FOV/Depth And PRO No-Crop (TAP-0043)
- Status: `Draft — not executed`
- Related Delivery: `TAP-0012`
- Contract: `ProductContract §3.1`

### Preconditions

- Use a tripod, stable light, center/edge landmarks, and near/far objects that
  reliably yield depth.
- Export original HEIC, manifest/crop/source/zoom facts, and Apple depth.
- Freeze the preview-to-output mapping and tolerance before execution; do not
  loosen it after seeing results.
- **Owner live:** compare preview and final composition.

### Reset And Install

1. Install the frozen build, complete setup, select default HEIC, reset Basic
   EV, disable scene-changing flash behavior, and begin clean logs.
2. Record the device capability matrix and every visible Standard FOV.
3. Do not move the camera or scene until all comparisons finish.

### Procedure

1. Capture a reference screenshot of the default Standard preview and FOV list.
2. For every visible Standard FOV: select and stabilize it; capture the preview;
   record RGB/depth source, raw zoom, and generation; take a photo; export
   original/manifest/depth facts; compare final RGB landmarks under the frozen
   mapping; then compare RGB/depth landmarks in the 2D depth view.
3. Rapidly traverse all Standard FOVs in both directions and capture once more;
   the artifact must use the final generation and plan.
4. **Owner live:** enter PRO after frosted readiness. Confirm the selector is
   hidden and the view is fixed rear LiDAR 24 mm / 1x.
5. Take at least three PRO photos in the same scene. Each must report the fixed
   source, raw zoom 1.0, full-frame/no product crop, aligned RGB/depth, and the
   matching manifest facts.
6. Return to Standard and verify the prior Standard selection/mapping resumes
   without PRO source or crop residue.

### Expected Results

- Every displayed Standard FOV resolves to an explicit RGB/depth capture plan;
  preview and output correspond under the frozen mapping and RGB/depth
  landmarks align.
- Rapid switching cannot write a stale source, zoom, or generation.
- PRO is always the fixed eligible 24 mm / 1x full-frame path, never a cropped
  or multi-focal product surface.

### Required Evidence

- Per-FOV preview, original HEIC, manifest/source/zoom/crop summary, depth facts,
  RGB/depth landmark comparison, and generation log.
- Three equivalent PRO evidence packages, Standard↔PRO recording, and the full
  result matrix.
- **Owner live:** written acceptance of preview/final composition and no-crop.

### Verdict

- **Pass:** every FOV actually displayed by the chosen device passes and all
  fixed PRO source/zoom/crop conditions hold.
- **Fail:** a visible FOV has no valid RGB/depth plan; preview/output exceeds the
  frozen mapping; stale plan data lands; Standard silently uses the PRO path; or
  PRO crops/zooms/exposes multiple focal choices.
- **Blocked:** a suitable device/scene, originals/manifest/depth, or pre-frozen
  mapping/tolerance is unavailable. A FOV not offered by that device is recorded
  as unavailable, not silently passed.

<a id="tap-0044"></a>

## Live Photo Capture, Signing, Readback, Audio, And Playback (TAP-0044)
- Status: `Draft — blocked until the owner freezes the matrix and run budget`
- Related Delivery: `TAP-0056`
- Contract: [ProductContract §3.2 and §5.2](ProductContract.md),
  [shared artifact contract index](CONTRACTS.md)

This is an attended physical-device procedure for the current Live Photo
chain. It does not test or imply per-frame MOV depth.

Before execution, the product owner must freeze the supported physical
device/iOS rows, HEIC/JPG and audio-state coverage, repetitions, maximum capture
count, artifact retention, and production-or-development App Attest
environment. An Agent must not choose those values after seeing the result.

### Preconditions

- The candidate build includes the current Live Photo capture, Pending Capture
  Queue signing/export, Photos original-resource loading, TAP Library, and
  native Live Photo playback paths.
- The selected device supports the candidate build's Live Photo path. Camera,
  Photos, and, for audio-captured rows, Microphone authorization are available.
- OS Microphone authorization and TAPCam's Microphone data-use preference can
  be controlled independently. Both must permit audio for an
  `audio = captured` row.
- The run has a visible scene change and a short audible cue that make motion
  and sound observable without identifying a person or recording private
  speech.
- Public-safe logs can correlate capture, paired-MOV delivery, manifest/proof,
  queue state, export, Photos resource loading, and playback without exposing
  raw paths, full identifiers, key IDs, assertions, or proof bodies.
- An approved diagnostic/export path can preserve or hash the exact signed
  primary photo and paired MOV before queue cleanup, then read the original
  Photos `.photo` and `.pairedVideo` resources without using a generic share
  path that may transcode them.
- A usable App Attest capture credential exists for the environment chosen by
  the owner. Production attestation itself is accepted separately under
  `TAP-0046`.
- **OWNER-LIVE:** approve the run budget and fill every selected row below.
  An unselected row must say `Not in approved matrix`, not `Passed`.

### Matrix To Freeze

| Row | Primary format | Microphone OS authorization | TAPCam audio data use | Repetitions | Owner decision |
| --- | --- | --- | --- | --- | --- |
| L1 | HEIC | Granted | On | `OWNER-LIVE` | `OWNER-LIVE` |
| L2 | HEIC | Granted or not granted | Off | `OWNER-LIVE` | `OWNER-LIVE` |
| L3 | JPG | Granted | On | `OWNER-LIVE` | `OWNER-LIVE` |
| L4 | JPG | Granted or not granted | Off | `OWNER-LIVE` | `OWNER-LIVE` |

The owner may reduce or expand this proposal before the run, but the frozen
matrix and reason must be recorded in the result. A row cannot change after its
first capture.

### Reset / Install Procedure

1. **OWNER-LIVE:** approve installation of the frozen signed build and any
   removal of an earlier test build. Do not delete unrelated Photos assets.
2. Record commit, build number, distribution method, signing environment,
   device model, iOS version, free storage, and current Camera/Photos/
   Microphone authorization.
3. Archive prior logs and test artifacts. If named test assets must be removed,
   show the exact list to the owner and use the system Photos confirmation.
4. Install or update using the approved method. Launch once, complete required
   setup, and confirm TAP Library is readable.
5. Set the first frozen matrix row. Start a continuous screen recording and
   public-safe device logs before entering Live Photo capture.

### Procedure And Expected Results

Run the numbered steps for every frozen matrix row and repetition.

| Step | Action | Expected result | Observed result | Evidence |
| --- | --- | --- | --- | --- |
| 1 | Record the row ID, output-format setting, both Microphone states, capture mode, and pre-capture queue count. | The recorded settings exactly match the frozen row. Changing OS authorization alone does not override a saved TAPCam data-use opt-out. | Pending | Pending |
| 2 | **OWNER-LIVE:** frame the approved moving scene, produce the approved audible cue for audio-on rows, and press the shutter once. | The shutter responds once, the live camera remains interactive, and exactly one candidate capture enters the pipeline. | Pending | Pending |
| 3 | Observe photo completion and the Apple movie-complement callback. | A successful Live Photo row produces one primary HEIC/JPG and one `paired-video.mov`. If the complement fails, the app may produce a valid still-photo v1 artifact, but it must not label or export a partial Live Photo v1 artifact; that row has not passed Live Photo acceptance. | Pending | Pending |
| 4 | Before cleanup, inspect the pending resource inventory and hash the exact resources through the approved diagnostic path. | The bundle has the selected primary container and exactly one paired MOV. No decoded frame, preview image, or adjusted Photos resource replaces either source. | Pending | Pending |
| 5 | Decode the embedded manifest from the primary photo. | The manifest passes the current shared Live Photo contract, reports the observed audio and primary-photo depth facts truthfully, and makes no per-frame MOV-depth claim. | Pending | Pending |
| 6 | Allow the Pending Capture Queue to sign the capture and inspect the resulting proof through the approved audit tool. | The exact photo/MOV pair passes the complete shared producer-binding checks and one App Attest capture assertion is written without changing the bound resources. | Pending | Pending |
| 7 | Observe the final local gate and Photos commit. | `validateSignedExportLivePhoto` accepts the exact signed photo/MOV pair before `saveDepthLivePhoto` can commit it; any shared-contract mismatch is rejected before export. | Pending | Pending |
| 8 | Load the saved asset's original Photos resources using the approved original-resource path. | Photos exposes the original `.photo` and `.pairedVideo`, and the complete shared local binding reproduces from those resources. Adjustment or presentation resources, if any, are recorded separately and never substituted. | Pending | Pending |
| 9 | Open the saved item in TAP Library, page away and back, then press and hold the Live Photo. | The correct item remains selected, native Live Photo motion plays without a blank replacement or crash, and release returns to the still presentation. Record the Viewer mute policy separately; this step does not silently convert captured-audio acceptance into a requirement for surprise sound in TAP Library. | Pending | Pending |
| 10 | Inspect the original paired MOV's audio tracks, then **OWNER-LIVE** plays it on the approved sound-enabled diagnostic or system surface. | Audio-on rows contain the captured audio track, report `audio = captured`, and reproduce the approved cue when sound is explicitly enabled. Audio-off rows contain no captured audio, report `audio = not-captured`, and remain silent. Ambient privacy-sensitive audio is not retained as acceptance evidence. | Pending | Pending |
| 11 | Export verification originals through the approved TAP verification-resource path and compare them with the Photos readback. | The primary file and paired MOV remain byte-identical to the Photos readback; transport routing metadata remains unsigned and supplies no trust facts. | Pending | Pending |
| 12 | Repeat steps 1–11 for the frozen matrix without changing counts or pass thresholds. | Every selected row produces a complete result record. A failed repetition remains in the report; it is not overwritten by a later success. | Pending | Pending |

### Required Evidence

- Frozen build/commit, distribution/signing environment, device/iOS matrix,
  row selections, repetitions, run budget, and operator.
- Continuous owner-attended recordings for capture and native playback, with
  privacy-sensitive audio excluded from the retained package.
- Per-capture public-safe event timeline: shutter, primary photo, movie
  complement, queue states, signing, final gate, Photos commit/readback, and
  playback.
- Primary format, Photos resource-type inventory, shared-contract conformance
  report, and redacted App Attest proof summary.
- SHA-256 audit table for the exact pre-export resources and Photos original
  readback. Raw media and proof material must remain in the owner-approved
  access-controlled evidence location; the Markdown result stores only
  redacted identifiers and hashes.
- Audio track/manifest result and owner-heard playback on the approved
  sound-enabled surface for every selected audio state; record TAP Library's
  mute policy separately.
- **OWNER-LIVE:** written confirmation of the visible motion, audio behavior,
  TAP Library paging/playback, and final matrix result.

### Verdict Conditions

- **Pass:** every frozen row and repetition completes the primary-photo plus
  paired-MOV chain; shared Live Photo contract facts are truthful; signing,
  final gate, Photos original readback, and native motion playback all succeed;
  the original MOV's audio behavior matches both permission layers on the
  approved sound-enabled surface; and the owner accepts the result.
- **Fail:** a selected row loses or substitutes an original resource; exports a
  partial Live Photo; binds the wrong bytes or schema; misreports audio or depth
  scope; bypasses the final gate; cannot reproduce the signed hashes from
  Photos originals; produces a missing/wrong audio track or cannot reproduce
  the cue on the approved sound-enabled surface in an audio-on row; captures
  audio in an audio-off row; or fails owner-observed native motion playback.
- **Blocked:** the build, matrix, repetitions, run budget, device, permission
  state, credential environment, safe diagnostics, exact-resource readback, or
  owner attendance is unavailable. Simulator, generated fixtures, or generic
  Photos sharing alone cannot establish this acceptance.

<a id="tap-0045"></a>

## TAP Video Device, Zero-Depth, Performance, And iCloud (TAP-0045)
- Status: `Draft — blocked until matrix and budgets are approved`
- Related Delivery: `TAP-0011`, `TAP-0057`
- Contract: `ProductContract §3.3`

This procedure does not invent performance targets. Before execution, the owner
must approve the supported device/iOS/audio matrix, iteration counts, CPU/RSS/
dirty-memory/disk/thermal/drop and codec p50/p95 budgets, iCloud-only method,
and registration tolerance. The historical registration candidate is at most
one display pixel; it is not active until approved for this run.

AirPlay, Picture in Picture, and background playback are explicit non-goals and
are neither steps nor failure conditions.

### Preconditions

- A candidate build includes the zero-depth correction and current TAP Video
  capture/sign/export/readback/playback chain.
- Each device has enough storage/power and a recorded initial thermal state.
- Logs expose recorder/drop/writer/finalization, tracks/KLV, manifest/proof,
  export/readback, RSS/thermal, codec, and registration facts.
- Real iCloud-only media is available; inability to make an original genuinely
  remote is Blocked, not simulated success.
- Zero-depth uses a reproducible real scene or an owner-reviewed device-only
  hook that keeps true RGB/audio. Writer failure uses a reviewed safe hook.
- **Owner live:** attend capture, transitions, zero-depth, playback, and iCloud
  interaction.

### Reset And Install

1. Install the frozen signed build and record commit, device/iOS, signing
   environment, storage, memory, and thermal baseline.
2. Use a dedicated named test-media set; do not automatically delete Photos
   assets during acceptance.
3. Disable Low Power Mode and begin each performance group at the approved
   thermal state.
4. Run independent logs for Microphone data Off and On; On also requires OS
   authorization. Cold-launch cases must truly terminate the process.

### A. PRO Video Regression

For each approved LiDAR Pro and audio state, execute the owner-approved counts;
the current proposed minimum matrix is:

1. Cold launch → PRO → VIDEO → first recording: 10 cycles.
2. PRO Photo ↔ PRO Video: 20 cycles.
3. Standard VIDEO ↔ PRO VIDEO: 10 cycles.
4. Record for five seconds and stop: 10 cycles.
5. During recording: AF tap 10 times; first MF tap assist after re-entry 10
   times; full MF slider movement 10 times.
6. VIDEO → front → rear PRO: 10 cycles.
7. After every cycle, inspect frost, recording state, TAP Library entry, and the
   next recording readiness. Automation may repeat actions, but owner-observed
   physical execution is still required.

Expected: no first-recording starvation, zero-duration RGB, black screen,
wedge, false recording state, or missing prepared graph.

### B. Artifact And Writer Truth

1. For representative recordings inspect original MP4, manifest, proof,
   RGB/audio/KLV tracks, depth coverage, and drop counters.
2. Microphone Off produces no audio; On produces real audio; manifest and tracks
   agree.
3. Signed/exported/readback bytes bind correctly; the TAP Library poster and
   foreground RAW/registered 2D paths behave according to available facts.
4. Trigger one reviewed writer failure. The UI leaves recording immediately,
   the failed workspace never enters the Pending Capture Queue, the graph
   rebuilds, and the next recording succeeds.

### C. Zero-Depth Contract

1. Produce a valid RGB/optional-audio recording with zero stored depth.
2. **Owner live:** stop and observe one non-blocking Depth unavailable message.
3. Confirm the artifact is retained, the manifest reports zero/missing depth,
   and the record proceeds through Pending Capture Queue signing, Photos export,
   and original-resource readback.
4. **Owner live:** RAW playback works in TAP Library. 2D may be unavailable when
   depth/registration is absent, but the record must not become terminal.

### D. Duration, Codec, Memory, And Thermal

1. Under the approved matrix, record 15, 60, and 180 seconds at least three
   times per duration.
2. Capture RSS, dirty memory, disk bytes, RGB/depth/audio drops, compression
   ratio, encode p50/p95, and thermal state.
3. No RGB/audio drop may be hidden; every depth gap/drop is reflected in
   counters/manifest. Steady-state RSS must not grow linearly with duration and
   all values must meet the pre-approved budgets.
4. Run exact-byte codec round trip plus decode p50/p95 over the approved real
   depth corpus.
5. For representative 180-second media run RAW/2D play, seek, dismiss, and an
   approved symbolicated CPU trace.
6. Repeat open → 2D play → dismiss five times and capture a memgraph ownership
   report. Trace-file size is not heap evidence; whole-video `Data` and repeated
   viewer retention require explanation.

### E. Registration

1. Capture known landmarks and inspect the same-frame RGB/registered-depth
   display.
2. Measure display-space landmark error against the tolerance approved before
   execution. Do not revise the tolerance after observing the result.

### F. Real iCloud-Only Lifecycle

1. Prove a signed TAP Video original is not local.
2. **Owner live:** open it. Poster/low-resolution preview remains visible and
   progress is singular and non-regressing.
3. Swipe away and return during download; stale callbacks cannot replace the
   current item. Dismiss during download; the request cancels with no later UI
   mutation.
4. Reopen, finish download, and confirm RAW plus registered 2D when available.
5. Produce a real offline terminal download error, restore connectivity, and
   exercise the visible recovery path.

### Required Evidence

- Per-device/audio cycle tables and all recorder/drop/writer/finalization logs.
- Representative MP4/manifest/proof/readback audits and complete zero-depth
  state transition plus recording.
- Raw 15/60/180-second metric samples, codec corpus/results, symbolicated CPU
  trace, memgraph ownership report, and registration measurement.
- Proof of real iCloud-only state plus progress/cancel/stale-callback recovery
  recording.
- **Owner live:** written acceptance of capture, transition, zero-depth message,
  playback, iCloud UX, and final metric report.

### Verdict

- **Pass:** the approved matrix and budgets complete with zero regression-cycle
  failures, end-to-end non-blocking zero-depth, truthful artifacts, acceptable
  resource behavior, registration, and iCloud lifecycle; the owner accepts.
- **Fail:** starvation/black screen/wedge/false recording; zero-depth discard or
  terminal state; hidden media facts; linear or over-budget resources;
  unexplained ownership; registration beyond tolerance; or regressing,
  uncancellable, stale iCloud updates.
- **Blocked:** any matrix/budget is unapproved; required device/corpus/storage/
  iCloud state/hook/diagnostics is missing; the owner cannot attend; or only
  Simulator/automation evidence is available.

<a id="tap-0046"></a>

## Production App Attest And Final Signed-Export Gate (TAP-0046)
- Status: `Draft — blocked until production scope, matrix, and budget are approved`
- Related Delivery: `TAP-0054`, `TAP-0056`, `TAP-0057`
- Contract: [ProductContract §3.4, §6, §7, and §8](ProductContract.md),
  [App Attest backend contract](BackendContract.md), and the shared
  [binding/proof contract](bindings/capture-binding-and-proof-v1.md)
- Production Backend Window: `OWNER-LIVE: approve operator, account, request budget, and retention before execution`

This procedure validates the production chain in four separate layers:

1. the installed build carries the production entitlement and runtime metadata;
2. the production backend accepts or recognizes the App Attest credential;
3. the device creates an offline capture assertion over the canonical media
   binding; and
4. the exact artifact passes the local fail-closed gate before Photos export,
   then remains valid on original-resource readback.

A backend `valid` response alone is not a verdict about a file. The acceptance
harness must first pass the shared local artifact-binding gate. This diagnostic
verification does not add a user-facing Verify action for TAPCam-owned captures.

### Preconditions

- **OWNER-LIVE:** authorize use of the production App Attest environment,
  production backend, selected test account or install, exact request/capture
  budget, change window, evidence retention, and cleanup policy.
- A frozen Release or TestFlight `.app` and its signed archive are available.
  Source build settings or an entitlement source file alone are insufficient;
  the installed signed product must be inspectable.
- The approved physical device reports App Attest support. Simulator or a test
  signer cannot satisfy this procedure.
- A production backend operator can correlate a redacted test run and confirm
  challenge/registration or credential reuse, accepted environment/app
  identity, active key status, and capture-signature verification without
  exporting secrets or raw backend bodies.
- The owner chooses one sanctioned credential branch before installation:
  **fresh registration** or **reuse existing ready credential**. Local
  `reset(photo_keyid)` does not delete an Apple private key or backend record,
  so it must not be presented as production cleanup.
- Public-safe device/backend logs and a locally trusted acceptance verifier are
  ready. Logs must not expose full credential names derived from users, key IDs,
  assertion objects, proofs, URLs with private paths, media bytes, or response
  bodies.
- The verifier can run the shared local artifact-binding gate for every frozen
  artifact class before submitting the unchanged
  `/tapcam/capture-signatures/verify` request.
- An owner-reviewed mutation harness can alter a disposable copy outside its
  proof slot without touching a Photos asset or production source artifact.
- Camera and Photos permissions are already available. Media-feature
  acceptance beyond the signing/export boundary remains in `TAP-0044` and
  `TAP-0045`.

### Representative Matrix To Freeze

| Row | Artifact | Required representative boundary | Repetitions | Owner decision |
| --- | --- | --- | --- | --- |
| A1 | HEIC still | Current shared Still Photo contract and photo final gate | `OWNER-LIVE` | `OWNER-LIVE` |
| A2 | JPG still | Current shared Still Photo contract and photo final gate | `OWNER-LIVE` | `OWNER-LIVE` |
| A3 | Live Photo | Owner-selected HEIC/JPG primary, original paired MOV, and Live Photo final gate | `OWNER-LIVE` | `OWNER-LIVE` |
| A4 | TAP Video | Owner-selected approved depth/audio state, video final gate, and original-video readback | `OWNER-LIVE` | `OWNER-LIVE` |

The four artifact classes are the proposed representative minimum. The owner
must freeze inclusion, repetitions, capture budget, and the exact A3/A4 media
state before the run. A missing decision is Blocked, not an Agent default.

### Reset / Install Procedure

1. **OWNER-LIVE:** approve the production window and artifact/request budget.
   Record the backend operator and stop conditions before any production call.
2. Archive prior evidence. Do not delete backend credentials, Keychain items,
   app data, or Photos assets unless the owner names the exact target and the
   production operator approves the corresponding cleanup procedure.
3. Record commit, build/archive identifier, distribution method, bundle/app
   identifier, device/iOS, and the selected fresh-or-reuse credential branch.
4. Extract the entitlements from the signed `.app` using the approved code-sign
   inspection tool and retain the output. Confirm the evidence belongs to the
   exact binary that will be installed.
5. Have the backend operator prepare redacted correlation for the selected
   account/install and record the pre-run credential state without exposing
   the full key ID or server secrets.
6. Install the frozen build through the approved Release/TestFlight path. Start
   public-safe device and backend logging before credential preparation.
7. If the owner selected fresh registration, use only the approved application
   and backend reset/registration procedure. If reuse was selected, do not
   manufacture a fresh registration merely to make the trace look complete.

### Procedure And Expected Results

| Step | Action | Expected result | Observed result | Evidence |
| --- | --- | --- | --- | --- |
| 1 | Inspect the entitlements extracted from the installed/frozen signed product. | `com.apple.developer.devicecheck.appattest-environment` is `production`, and the signed product's app identifier matches the production backend expectation. A source-only entitlement value is not accepted as evidence. | Pending | Pending |
| 2 | Inspect the runtime's public-safe configuration summary and the backend target selected by the build. | Release/TestFlight selects `https://www.tapnap.net` with App Attest metadata `.production`; entitlement, runtime metadata, bundle/app identity, and backend environment agree. No development or localhost endpoint is used. | Pending | Pending |
| 3 | **OWNER-LIVE:** enter the camera or use the Release `Photo Integrity` Prepare/Retry action according to the approved branch, then wait for the preparation result. | Fresh branch: the backend issues and atomically consumes an attestation challenge, validates Apple's attestation/app/environment/public key/initial counter, and accepts the credential before local persistence. Reuse branch: the existing ready `photo_keyid` mapping resolves to an active matching production credential. Release UI reaches `Ready` only after its assertion health check; it exposes no backend details or full key ID. | Pending | Pending |
| 4 | Backend operator correlate the preparation trace with the device trace. | Exactly the approved branch occurred. Credential name remains a lookup name, not a user identity or trust claim. The backend, not the client-ready flag, is the trust decision point. | Pending | Pending |
| 5 | For each frozen A1–A4 row, capture one approved test artifact and let the Pending Capture Queue process it. | The record moves through the applicable pending/signing/signed/exporting/exported/readback states without an unsigned fallback or silent trust downgrade. Counts stay within the approved production budget. | Pending | Pending |
| 6 | Audit the proof generated for every artifact before Photos export. | The device uses the registered production key and passes the complete shared producer-signing and family-binding checks. Offline capture signing requests no online business-request assertion challenge. | Pending | Pending |
| 7 | Observe the final local export gate for A1–A4. | HEIC/JPG pass `validateSignedExportPhoto`; Live Photo passes `validateSignedExportLivePhoto` for the exact primary/MOV pair; TAP Video passes `validateSignedExportVideoFile` for the exact MP4. Each gate recomputes byte binding and checks expected identity/proof before Photos commit. Queue status or a filename never substitutes for validation. | Pending | Pending |
| 8 | Read the saved Photos originals through the approved resource APIs and run the same applicable binding check. | HEIC/JPG originals retain the signed container/proof; Live Photo exposes the bound original `.photo` plus `.pairedVideo`; TAP Video streams its original video resource to disk-backed validation. The recomputed binding still matches, with no compatibility export or re-encoding substituted. | Pending | Pending |
| 9 | In the acceptance verifier, run the shared local artifact-binding gate over the exact readback bytes, then submit the shared verification request to the production endpoint. | Local file/resource checks pass before the request. The backend confirms that the registered active production key signed the submitted binding and returns the contracted `valid` result. No media, manifest, depth data, or recomputed resource hash is uploaded. | Pending | Pending |
| 10 | **OWNER-LIVE:** inspect TAPCam's Release presentation for the captured assets and Share entry. | TAPCam displays persisted protection/verifiability state for its own completed captures and does not trigger a new product Verify operation merely because an item is viewed or shared. The explicit backend calls in step 9 remain acceptance diagnostics. | Pending | Pending |
| 11 | With the owner-reviewed harness, alter one disposable still/Live primary copy outside its proof slot, alter one disposable paired MOV or TAP Video copy, and run local verification before any backend request. | Every altered copy fails local content-binding/final-gate validation. It is not exported and cannot be called verified even if an unchanged embedded assertion would still be cryptographically valid for its original binding. | Pending | Pending |
| 12 | Review request counts, credential state, Photos assets, and retained evidence with the owner and backend operator. | Counts remain within the approved budget; cleanup follows the pre-approved policy; failures and partial traces remain recorded; secrets and private media are not copied into the public Markdown report. | Pending | Pending |

### Required Evidence

- Frozen commit/build/archive, installed-product identity, distribution method,
  device/iOS, owner-approved production window, request/capture budget, and
  fresh-or-reuse decision.
- Entitlement extraction from the exact signed product plus public-safe runtime
  environment/backend summary.
- Redacted device/backend correlation for credential preparation, accepted app
  identity/environment, active credential status, and the Release readiness
  result. Store raw production logs only in the approved restricted location.
- A1–A4 matrix with queue state timeline, shared-contract conformance report,
  expected capture/package identity, final-gate outcome, Photos original-resource
  inventory, and readback outcome.
- Local binding-reconstruction report and redacted production verification
  response for every representative artifact. The report must show that local
  validation preceded the backend request.
- Mutation-test input hashes and rejection stages for the disposable copies;
  do not retain mutated copies in Photos.
- Public-safe request-count and cleanup report. Full key IDs, assertion
  objects, proof bodies, backend response bodies, raw private URLs, and private
  media do not belong in the Markdown record.
- **OWNER-LIVE:** written acceptance of the production environment alignment,
  Release readiness presentation, four-row result, negative mutation behavior,
  and cleanup report.

### Verdict Conditions

- **Pass:** the exact installed product carries the production entitlement and
  selects matching production runtime/backend metadata; the approved fresh or
  reuse credential branch is accepted by the backend; every frozen HEIC, JPG,
  Live Photo, and TAP Video row contains a real production App Attest capture
  assertion, passes the correct pre-export and Photos-readback binding gates,
  and receives backend `valid` only after local reconstruction; altered copies
  fail locally; budgets and privacy rules hold; and the owner accepts.
- **Fail:** development/production identity mismatch; client-only readiness is
  treated as backend trust; attestation/credential rejection; a missing,
  wrong-key, wrong-identity, or wrong-binding assertion; unsigned fallback;
  Photos export before the applicable final gate; readback mismatch; media
  uploaded to the signature endpoint; backend `valid` used despite a local
  media mismatch; mutation accepted; secret leakage; or any unrecorded
  representative-row failure.
- **Blocked:** production authorization, operator, account/install choice,
  request/capture budget, frozen build, signed entitlement evidence, supported
  physical device, backend observability, exact-resource readback, local
  verifier, mutation harness, safe evidence storage, or owner attendance is
  missing. Development App Attest, Simulator, test-signature fixtures, and
  source inspection alone cannot establish this acceptance.

### Human Confirmation

`Pending — OWNER-LIVE must approve the production run and accept its retained result.`

<a id="tap-0047"></a>

## TAP Library Permission, Delete, And Return Semantics (TAP-0047)
- Status: `Draft — not executed`
- Related Delivery: `TAP-0058`, `TAP-0059`, `TAP-0061`
- Product Contract: `ProductContract §5.1–5.4`; `§8`

This is an attended physical-device procedure. It covers Limited Photos access,
source-owned deletion, post-delete Viewer selection, empty-Library close, and
in-session return in a large Library. It does not redefine the TAP Library as
the private Pending Capture Queue, and it does not promise durable pixel-exact
scroll restoration across a fresh Library entry.

The owner must approve the disposable media set, visible item order, large-
Library fixture, device/iOS matrix, and any method used to hold a capture in a
genuine pending/local-only state. No personal asset is a test deletion target.

### Preconditions

- The frozen candidate build implements the mixed-media TAP Library, horizontal
  Viewer, current bottom toolbar, source-owned deletion, and route bookmarks.
- The owner has approved a disposable Photos set containing:
  - app-owned exported assets that may be deleted;
  - an allowed subset and an excluded subset for Limited access; and
  - enough ordered items to exercise middle, terminal, and single-item deletion.
- A genuine pending or local-only capture can be prepared through the normal
  product path or an owner-reviewed test path. The method must not relabel an
  exported Photos asset as pending and must not mutate the artifact under test.
- The owner has approved the large-Library fixture and has recorded its item
  count and canonical sort order. This draft intentionally defines no minimum
  count or scroll distance.
- Screen recording, public-safe app logs, and before/after Photos screenshots
  are ready. Private Photos identifiers remain in the evidence bundle and are
  not copied into public logs or this report.
- **OWNER-LIVE:** approve the disposable asset list, Photos authorization
  changes, every destructive confirmation, and the pending-item preparation
  method before the run starts.

### Reset / Install Procedure

1. Archive previous evidence and record the current Photos authorization state.
2. Install the frozen signed build using the owner-approved install method.
3. Using the system-owned Photos access UI, grant TAPCam Limited access to only
   the approved subset. Record the selected and intentionally excluded fixture
   assets without exposing unrelated personal media.
4. Prepare the approved pending/local-only item and confirm it has not exported
   to Photos before its deletion steps.
5. Load the approved large-Library fixture. Record the visible canonical order
   and the identities of the disposable middle, terminal, and single-item test
   targets.
6. Begin continuous screen recording and app/device logs before opening TAP
   Library.

If Limited authorization, a genuine pending item, or a disposable ordered
fixture cannot be established, stop as Blocked instead of substituting a Web or
Simulator fixture.

### Procedure And Expected Results

| Step | Action | Expected result | Observed result | Evidence |
| --- | --- | --- | --- | --- |
| 1 | **OWNER-LIVE:** Open TAP Library while Photos access is Limited. Compare the grid with the owner-approved allowed and excluded subsets. | TAP Library loads without an app-owned replacement permission prompt. It may show accessible app-owned/pending content, but it must not reveal an excluded Photos asset or claim full-library access. | `Pending` | `Pending` |
| 2 | Leave and re-enter TAP Library once without changing the system selection. | The authorization remains Limited, the same permitted set is used, and re-entry does not silently broaden access or request permission again. | `Pending` | `Pending` |
| 3 | Change the Limited selection through the system-owned access UI using the owner-approved add/remove fixture, then return to TAPCam. | TAP Library refreshes to the newly authorized set without duplicating items, exposing removed assets, or reopening first-install setup. | `Pending` | `Pending` |
| 4 | Open an exported Photos asset, tap Delete, and cancel at the system confirmation. | TAPCam presents no app-owned confirmation before the Photos prompt. Cancel leaves the Photos asset, grid item, Viewer item, and current position intact. | `Pending` | `Pending` |
| 5 | **OWNER-LIVE:** Delete an approved exported Photos asset that is in the middle of a recorded three-or-more-item order, and confirm only the system prompt. | Exactly one system-owned confirmation is used. The deleted asset leaves Photos/TAP Library, and the Viewer selects the item that occupied the next index in the pre-delete order. | `Pending` | `Pending` |
| 6 | Delete the terminal exported Photos asset in the controlled order and confirm the system prompt. | With no next item, the Viewer selects the previous remaining item. It does not close while any TAP Library item remains. | `Pending` | `Pending` |
| 7 | Open the genuine pending/local-only item, tap Delete, and cancel the app-owned confirmation. | One TAPCam-owned confirmation appears, no Photos system deletion prompt appears, and cancel preserves the local record and artifact. | `Pending` | `Pending` |
| 8 | **OWNER-LIVE:** Delete the same pending/local-only item and confirm the app-owned prompt. | The Pending Capture Queue removes the intended local record through its normal storage boundary. No Photos system deletion prompt appears, and adjacent selection follows the same next-then-previous rule. | `Pending` | `Pending` |
| 9 | Establish the approved one-visible-item fixture, open its Viewer, and delete it using the confirmation owned by its actual source. | After the last visible item is removed, the Viewer closes to the empty TAP Library. It does not remain on stale content or an invalid next/previous route. | `Pending` | `Pending` |
| 10 | In the approved large Library, scroll well beyond the initial viewport, record a distinctive allowed item and its viewport position, open it, page left/right once if valid, return to the originally selected item, then return to the grid. | The in-session return lands at or near the clicked item using its item bookmark; the item remains recognizable without a top-of-grid jump or unrelated selection. Exact pixel offset is not promised. | `Pending` | `Pending` |
| 11 | Leave TAP Library completely, then open it as a fresh entry. | Fresh entry follows the current top-start contract. The prior view-local offset is not treated as a durable cross-entry coordinate. | `Pending` | `Pending` |

### Required Evidence

- Logs: authorization-state changes, Library snapshot/merge counts, selected
  source type, deletion request/result, pending-store removal, post-delete
  selected index, Viewer close, and route-bookmark resolution.
- Screenshots/recording: continuous attended recording of Limited selection,
  both confirmation owners, cancel/confirm outcomes, adjacency, empty close,
  and large-Library return.
- Output artifacts: before/after inventory of only the disposable Photos and
  pending/local targets, kept in the private evidence bundle.
- Result bundles: frozen build/commit, device/iOS, fixture manifest, approved
  sort order, and any focused automated result used as supporting evidence.
- **OWNER-LIVE:** written confirmation that no personal asset was targeted,
  each prompt had the correct owner, and the observed return position was
  acceptable for the approved large-Library fixture.

### Verdict Conditions

- **Pass:** every approved fixture completes; Limited access never exposes an
  excluded asset; exported and pending/local deletion use exactly their owning
  confirmation; cancel is non-destructive; next/previous/empty behavior is
  correct; in-session return preserves the clicked-item context; and the owner
  accepts the run.
- **Fail:** TAPCam adds a second confirmation for Photos deletion, omits its
  confirmation for local deletion, deletes on cancel, deletes the wrong item,
  exposes an excluded asset, duplicates/stales the Limited set, selects the
  wrong adjacent item, leaves an empty Viewer open, or loses the approved
  in-session clicked-item return.
- **Blocked:** the owner has not approved the device, disposable fixtures, or
  destructive steps; Limited access cannot be established; a genuine
  pending/local item cannot be held; canonical order cannot be recorded; logs
  cannot distinguish Photos from local deletion; or only Web/Simulator
  fixtures are available.

<a id="tap-0048"></a>

## Approved Web Prototype To SwiftUI Parity (TAP-0048)
- Status: `Evidence in progress — startup prototype approved; complete Simulator parity matrix remains pending`
- Related Delivery: startup `TAP-0008`, `TAP-0009`, and reviewer candidate
  `TAP-0087`; Share `TAP-0081`; prototype foundation `TAP-0006`; repository
  relocation `TAP-0096`
- Product Contract: `ProductContract §5.2–5.3, §6, §9`
- Prototype Tasks: `TAP-0008`, `TAP-0009`, `TAP-0081`, `TAP-0087`
- Prototype Repository Task: `TAP-0096`
- Prototype Repository/Path/Revision: sibling `TAPCamPrototype` repository;
  `Prototype/startup-lifecycle.html` is current `TAP-0087-r1-candidate` v19 and
  `Prototype/index.html` retains the independently approved TAP-0008-r2,
  TAP-0009-r1, and TAP-0081-r3 slices
- Prototype Owner Approval: TAP-0008-r2, TAP-0009-r1, TAP-0081-r1,
  TAP-0081-r3, and the complete `TAP-0087-r1-candidate` v19 composition are
  `ownerApproved`; TAP-0087 approval was recorded on 2026-09-04
- Share Build/Commit: `f2bf8d2` (`Close TAP-0081 and TAP-0082 Share lifecycle`);
  startup native candidate must be frozen when the remaining matrix is run
- Device/iOS: `TAP-0082 accepted on iPhone 15 Pro / iOS 26.6`
- Superseding Decision: the accepted `TAP-0082` run and its Git-retained acceptance
  record remove the old visible Cancel, 50 ms delayed reveal, and 400 ms minimum
  hold requirements
- Simulator Matrix: `Pending owner confirmation for the frozen candidate build`
- Date: `2026-09-04` (procedure synchronized; matrix not executed)
- Operator: `Codex development session /root; prototype approved, parity audit pending`

This record instantiates the Web-to-SwiftUI comparison for all prototype
surfaces currently assigned to TAP-0048: First-Install Setup, Required
Permission Check, Resource Initialization, and TAP Share. The owner has
accepted the physical-device Share flow under TAP-0082 and the complete
TAP-0087 v19 Web composition, while the full TAP-0048 Simulator state/viewport
comparison remains deliberately open.
A Web fixture can prove visual intent and simulated
state coverage only. It is never evidence that Camera, Photos, permissions,
depth, capture, signing, export, haptics, accessibility, performance, system
share destinations, or device lifecycle actually works.

The approved interaction is one stable app-owned surface anchored to Share.
Selecting an implemented format immediately replaces only that row's subtitle
with a thin determinate track in the same fixed-height slot. The selector,
title, icon, badge, row, popover, and sibling geometry remain mounted and
stable; no percentage or Cancel control appears. Public-safe failure and Retry
remain in the same anchored surface. As soon as the selected payload is ready,
the popover disappears and exactly one system-owned activity controller is
presented. The Web prototype deliberately stops at that native system boundary.

### Preconditions

- **Met:** the current sibling manifest at
  `../TAPCamPrototype/Prototype/manifest.json` records extraction
  provenance plus distinct current and historical axes. Its
  `acceptanceConsumers.TAP-0048` record names all three startup surfaces and
  their exact approval status. It also preserves the TAP-0081 revision data
  recorded by the historical
  [closure manifest at `f2bf8d2`](https://github.com/TAP-NAP/TAPCamDemo/blob/f2bf8d2dc5f10df3eee7f468ebc22b9e33fca22a/Prototype/manifest.json), including the
  `TAP-0081-r1` geometry baseline and freezes owner-approved
  `TAP-0081-r3-candidate` behavior across Photo/Live Photo/TAP Video fixtures,
  three credential states, selection, immediate same-slot progress,
  payload-ready handoff, failure, Retry, native symbol intent, responsive
  review viewport, and explicit non-goals. It explicitly records no percentage
  or Cancel, no artificial reveal delay, and no minimum progress hold.
- **Met — OWNER-LIVE:** the product owner approved exact revision
  `TAP-0081-r1` on 2026-08-12. An undated screenshot, local working copy, old
  Sketch file, or Agent-selected Web state is not the baseline.
- **Met:** the r3 native delivery is frozen at `f2bf8d2`; focused behavior/build and
  TAP-0082 physical-device acceptance are recorded by their owning Tasks.
- **Met — OWNER-LIVE:** on 2026-09-04 the product owner approved exact
  `TAP-0087-r1-candidate` v19, including Required Permission Check, and asked
  that TAP-0048 remain for a later parity pass.
- **Pending:** map every approved state to the complete owner-confirmed
  Simulator/viewport matrix and capture its side-by-side parity evidence.
- **Pending — OWNER-LIVE:** approve the Simulator/viewport/state matrix. This
  record intentionally invents no tolerance, pixel threshold, Dynamic Type
  matrix, locale matrix, or visual-diff budget.
- System-owned controls and native icon assets or SF Symbols are identified in
  the comparison sheet. Web rendering is an optical reference; the SwiftUI
  result retains native semantics and is not replaced with a fake Web-like
  control.
- Simulator screen capture and result collection must be ready before executing
  the remaining matrix. Physical-device acceptance is recorded by completed
  `TAP-0082`; this record does not manufacture additional device artifacts.

### Candidate SwiftUI Mapping

This mapping describes the implementation seam to inspect; it is not a parity
verdict.

| Approved responsibility | SwiftUI candidate | Evidence state |
| --- | --- | --- |
| First-Install Setup hierarchy, row-owned actions, required/optional presentation, and iOS-owned permission boundary | `WelcomeStartupSetupView`, `StartupRequirementRow`, and `StartupGateCoordinator` | `TAP-0008-r2 independently approved; current native parity cells pending` |
| Required Permission Check with Camera/Photos-only recovery | `RequiredPermissionCheckView` and `StartupGateCoordinator` | `TAP-0087-r1-candidate` v19 owner-approved; native parity cells pending |
| Resource Initialization stable overlay and Camera-plus-Library readiness handoff | `CameraInitialReadinessOverlayView`, `CameraView`, and `StartupInitializationStore` | `TAP-0009-r1 independently approved; current native parity cells pending` |
| Stable Share toolbar leaf shared by Photo, Live Photo, and TAP Video | `DepthViewerShareControl` mounted from the shared Viewer chrome | `Implemented in f2bf8d2; complete parity cell pending` |
| App-owned anchored selection/immediate same-slot preparation/failure surface | `DepthAnalysisSharePopover` using native popover adaptation and stable in-place state layers | `Implemented in f2bf8d2; complete parity cell pending` |
| Frozen media subject, credential state, immediate monotonic preparation, attempt-scoped discard/cleanup, Retry, stale-callback rejection, and payload lease | `DepthAnalysisShareCoordinator` | `Focused tests recorded by TAP-0081/TAP-0082; complete parity cell pending` |
| Exact Share/Delete template vectors, Back-matched circular backgrounds, compact centered mode capsule, and additive Share progress ring | `ViewerToolbarIconButton` plus `DepthViewerModeCapsule` in the shared Viewer chrome | `Owner device result accepted under TAP-0082; Simulator parity cell pending` |
| One subsequent native system destination chooser | sibling `VerificationExportActivityView` presentation after popover disappearance | `Owner device result accepted under TAP-0082; Simulator parity cell pending` |

The candidate must preserve the contract rather than merely resemble the Web
pixels: resources are prepared only after selection; Share never contacts the
TAP backend or repeats App Attest verification, while the frozen downloaded
    original must pass local proof/content binding before it is labeled Verified
    in this owned-capture Share flow; this state does not independently verify
    App Attest assertion authenticity without the backend-held public key;
an implemented format tap immediately shows monotonic determinate progress in
the selected row's existing subtitle slot, without a percentage, Cancel,
delayed reveal, artificial minimum hold, or geometry change; payload readiness
ends the app-owned popover immediately; and no package is pre-generated,
background-prewarmed, or retained as a persistent cache.

### Reset / Install Procedure

1. Confirm the sibling
   `../TAPCamPrototype/Prototype/manifest.json` names the current
   `TAP-0087-r1-candidate` v19 workbench, the three TAP-0048 startup surfaces,
   and each surface's approval status. The previously approved `TAP-0081-r1`
   remains the Share geometry baseline; exact
   `TAP-0081-r3-candidate`, including local-integrity states, refined toolbar
   geometry, immediate same-slot progress, no visible Cancel, and immediate
   payload-ready handoff, was owner-approved on 2026-08-14.
2. Produce read-only Web reference captures directly from that revision. Do not
   manually redraw or edit them after approval.
3. Freeze the SwiftUI candidate commit and record scheme, configuration,
   Simulator model, iOS runtime, and locale.
4. Reset/install that build on every owner-approved Simulator in the comparison
   matrix. The separate physical-device run has already completed under
   `TAP-0082`.
5. Prepare each deterministic SwiftUI credential/media/preparation state through
   a documented fixture or real runtime path. Label fixture-backed and runtime
   states separately.
6. Create one empty parity row for every approved state × Simulator viewport,
   then capture Web and SwiftUI results side by side.

### Procedure And Expected Results

| Step | Action | Expected result | Observed result | Evidence |
| --- | --- | --- | --- | --- |
| S1 | Freeze the exact current and historical prototype axes from `manifest.json`. | TAP-0008-r2 Setup and TAP-0009-r1 Resource Initialization retain their independent approvals; the complete `TAP-0087-r1-candidate` v19 composition, including Required Permission Check, is owner-approved. | `Pending` | `Pending` |
| S2 | Compare First-Install Setup across every owner-approved Simulator viewport and required status fixture. | SwiftUI preserves the approved row hierarchy, exact icon identities, row-owned explicit actions, required/optional distinction, and app/system boundary without treating historical `/healthz` wording as current authority. | `Pending` | `Pending` |
| S3 | Compare Required Permission Check across Camera/Photos authorized, denied, restricted, Settings-return, and recovery states. | Only Camera and Photos appear; row state and recovery match the current candidate, and no Network, timeout, skip, or invented app-owned system dialog appears. | `Pending` | `Pending` |
| S4 | Compare Resource Initialization across preparing, Camera-first, catalog-first, and ready handoff states. | The stable surface remains visible until real Camera and first usable Library-catalog readiness both succeed; no failure, Retry, timeout, degraded entry, or unrelated warm-up UI appears. | `Pending` | `Pending` |
| 1 | Verify the frozen prototype manifest against TAP-0081 and Product Contract §5.3/§6. | Every reviewed state has one revision, media kind, credential state, viewport, and contract mapping; no deprecated modal Share sheet or direct-share path is included. | `Pending` | `Pending` |
| 2 | Capture the approved geometry from `TAP-0081-r1` and the approved original-readiness/local-integrity and preparation states from `TAP-0081-r3-candidate`. | The references preserve the stable Viewer hierarchy while adding Share-disabled iCloud loading, the text-free local-check skeleton, Failed direct-media warning, and immediate same-slot progress with no visible Cancel. Both exact revisions are recorded as owner-approved. | `Pending` | `Pending` |
| 3 | Open Share from Photo, Live Photo, and TAP Video in the SwiftUI candidate. | The same native Share control and anchored popover appear for all three media kinds without remounting the pager, media surface, video player, toolbar, or chrome. | `Pending` | `Pending` |
| 4 | Exercise Viewer original loading plus Verified, Needs Retry, and Failed fixtures. | Share stays disabled until the complete local/iCloud original is ready. Opening Share freezes that resource and runs only local proof/content-binding validation; it never calls backend/App Attest Verify. Photos without a queue record can become Verified from embedded identity; Needs Retry remains queue-only. | `Pending` | `Pending` |
| 5 | Compare format rows, native icons, copy, enabled states, and Coming Soon boundaries for all media kinds. | Locally valid Still/Live expose TAPNAP Package and Share Image; a local mismatch disables Package but retains Image/Video with an explicit unverifiability warning. Video Package remains Coming Soon; Sticker and Link remain disabled. Native icon assets and SF Symbols match their respective manifest intent, including the exact Share/Delete vector geometry. | `Pending` | `Pending` |
| 6 | Run a fast preparation. | The selected row's subtitle is replaced immediately by the thin determinate track in the exact same slot, without percentage or Cancel; payload readiness ends the popover immediately. No row flash, blank frame, toolbar rebuild, full-screen loading state, delayed reveal, or artificial hold appears. | `Pending` | `Pending` |
| 7 | Run a deliberately slow preparation while recording progress updates. | The same-slot track is visible immediately, advances monotonically, and does not move the selector, title, icon, badge, row, popover, or siblings. Other options remain visible but disabled. There is no percentage, Cancel, delayed reveal, minimum-visible hold, or ready overlay; the Share glyph remains fixed and its ring is additive. | `Pending` | `Pending` |
| 8 | Exercise a controlled failure and Retry the same option; separately trigger stale/superseded attempt cleanup through the native test seam when available. | Failure remains in the anchored surface with public-safe copy and Retry restarts only the failed option. No user-visible Cancel state is introduced. A stale callback or prior artifact cannot affect the new attempt, and internal discard/cancellation remains attempt-scoped rather than becoming an approved visible control. | `Pending` | `Pending` |
| 9 | Complete a ready payload and dismiss the system presentation. | The app-owned popover disappears before exactly one native system activity controller appears. Dismissal returns to the unchanged Viewer; per-attempt temporary resources are cleaned. No persistent share cache is created. | `Pending` | `Pending` |
| 10 | Inspect localized copy, accessibility identifiers/values, Reduce Motion behavior, material, geometry, spacing, and native-platform variances on each approved Simulator viewport. | The native result preserves the approved component relationships and semantics; documented optical platform differences do not become Web imitation controls. | `Pending` | `Pending` |
| 11 | Record every mismatch and classify it as implementation defect, approved native-platform variance, prototype revision request, or Product Contract conflict. | No Agent silently changes the prototype or Product Contract; every accepted variance has explicit owner disposition. | `Pending` | `Pending` |
| 12 | **OWNER-LIVE:** review the final side-by-side Simulator matrix and exceptions. | The owner either accepts the exact r1-geometry/r3-behavior prototype plus build pair for TAP-0048 or keeps this parity Task Pending with named mismatches. The already accepted TAP-0082 device verdict remains separate evidence. | `Pending` | `Pending` |

### Required Evidence

- Logs: Share presentation/preparation/handoff state identifiers, monotonic
  progress, attempt-scoped stale/discard cleanup when exercised, and
  fixture-versus-runtime provenance for every compared SwiftUI state. No
  user-visible cancellation state is required or inferred.
- Screenshots/recording: `N/A` by default. Review Web and Simulator side by
  side live; retain images or recordings only if the owner first approves that
  exact evidence form in this procedure. TAP-0082 retains its own evidence
  policy separately.
- Output artifacts: prototype manifest and approval record, structured
  DOM/native geometry and state assertions, completed textual/JSON parity
  matrix, and documented exception dispositions.
- Result bundles: frozen prototype and code commits, Simulator UI test bundles
  where used, build/Simulator/iOS identifiers, and a link to the completed
  `TAP-0082` device-acceptance record.
- **OWNER-LIVE:** prototype revision approval is recorded; explicit acceptance
  of every native variance and the final SwiftUI implementation pair remains
  required.

### Verdict Conditions

- **Pass:** all cells in the owner-approved matrix match the approved visual and
  interaction relationships or have explicit accepted native variances; no
  Product Contract conflict remains; Simulator/runtime claims are bounded to
  the evidence actually gathered; the owner accepts the exact prototype/build
  revisions; and physical-device claims remain bounded to the accepted
  `TAP-0082` record rather than inferred from Simulator evidence.
- **Fail:** SwiftUI drifts from hierarchy, icon identity, relative geometry,
  navigation, or approved state presentation; deprecated UI returns; a mismatch
  is hidden by changing the baseline; a fake control replaces native behavior;
  or a Web/SwiftUI fixture is presented as runtime evidence.
- **Blocked:** the frozen implementation build is missing; the owner has not
  approved the comparison matrix or variance policy; a deterministic state
  cannot be reached; a platform conflict requires a product decision; required
  Simulator evidence is unavailable; or the owner cannot attend the audit.

### Human Confirmation

`Pending for the complete TAP-0048 Simulator parity matrix. Exact r1/r3 Web
revisions and native f2bf8d2 are frozen, and physical-device Share acceptance
is complete under TAP-0082; those facts do not silently complete this separate
matrix audit.`

<a id="tap-0049"></a>

## Lifecycle-Correct Locked Camera Experiment (TAP-0049)
- Status: `Draft — Blocked until TAP-0013 is ready`
- Related Delivery: `TAP-0013`
- Product Contract: `ProductContract §4`; `§8`
- Experiment Branch: `Dedicated branch required; to be recorded`
- Soak Duration/Repetitions: `Owner decision required before execution`
- Import Observation Window: `Owner decision required before execution`

This is the promotion evidence draft for a new lifecycle-correct experiment,
not an acceptance run for the existing POC. The old E1–E7 branches, routes,
timings, log results, and smoke counts are historical diagnostic input only.
They do not define the candidate, its baseline, its soak duration, or a passing
result.

Execution remains Blocked until `TAP-0013` delivers a candidate on a dedicated
experiment branch. Passing this procedure still does not make Locked Camera a
production capability; production promotion requires a separate owner decision.

### Preconditions

- The candidate identifies its dedicated experiment branch, frozen
  commit/build, bounded scope, tests, and remaining gaps. A main-tree historical E-series build is not eligible.
- The candidate uses only APIs exposed in the installed SDK's public Swift
  interfaces and public documentation. It does not dynamically call, link, or
  emulate private or `.tbd`-only transition symbols, including historical
  examples such as `openApplicationAfterTransitionCompletion(for:)`,
  `applicationDidCompleteTransition()`, `urlsToOpen`, or `hasActiveSession`.
- No private/programmatic dismiss mechanism is used. Exit and suspension follow
  the system-supported user or lifecycle path approved for this run.
- The extension owns only locked launch UI, the live camera/depth capture path,
  and writing the declared unsigned capture resource to its session-content
  boundary. It does not request permissions or run App Attest, network, Photos
  export, Live Photo, or Pending Capture Queue work.
- The containing app uses the public Locked Camera manager/update boundary to
  import exposed session content into the existing Pending Capture Queue. The
  exact candidate artifact layout is documented by `TAP-0013`; no old flat-HEIC
  or staging layout is assumed by this procedure.
- Structured, public-safe correlated logs cover control intent, extension scene
  and root, session setup/running/interruption, real first frame, shutter/write,
  suspend/exit, containing-app exposure/import, resource release, and relaunch.
- The owner has approved the physical device/iOS matrix, launch source, exit
  path, soak duration, capture schedule, repetition count, import observation
  window, and any recovery/interruption scenario. This draft inherits none of
  the historical five-minute/three-capture/three-relaunch suggestions.
- **OWNER-LIVE:** approve the branch/build, matrix, destructive install/reset,
  duration/repetitions, and exact system-supported lifecycle actions.

### Reset / Install Procedure

1. Archive old E-series logs separately and label them Historical; do not merge
   them into this candidate's result bundle.
2. Record the candidate branch, commit, build configuration, entitlements,
   public-API inventory, extension dependency audit, device/iOS, available
   storage, initial thermal state, and approved test schedule.
3. Install the frozen signed build using the owner-approved method. Complete any
   required containing-app preparation before locking the device; do not grant
   a new permission from inside the locked extension.
4. Confirm the real system Locked Camera control is available. Directly opening
   a debug view, SwiftUI preview, or Simulator scene is supporting evidence only
   and cannot replace the lock-screen launch.
5. Empty or inventory only the approved candidate test records. Do not delete
   unrelated Photos or Pending Capture Queue data.
6. Start synchronized screen recording plus control-extension, capture-
   extension, and containing-app logs before the first lock-screen action.

### Procedure And Expected Results

| Step | Action | Expected result | Observed result | Evidence |
| --- | --- | --- | --- | --- |
| 1 | Review the frozen binary/source dependency and API inventory before device execution. | Every Locked Camera call is public for the approved SDK/deployment target; no private, hidden, selector-based, or `.tbd`-only API is present; forbidden extension-side App Attest/network/Photos/queue dependencies are absent. | `Pending` | `Pending` |
| 2 | **OWNER-LIVE:** Lock the device and launch the candidate through the real system Locked Camera control. | Correlated logs show the real control intent, secure-capture scene, persistent root/controller, camera preparation, and a visible non-empty UI. No containing-app UI or debug direct-launch path substitutes for it. | `Pending` | `Pending` |
| 3 | Wait for the candidate's readiness gate without tapping the shutter. | A real preview frame is visibly presented and recorded by the first-frame signal before shutter/primary controls become safe. Session configuration or a placeholder image alone is insufficient; no unexplained black/frozen surface appears. | `Pending` | `Pending` |
| 4 | Capture according to the owner-approved schedule while observing the live surface. | Each shutter action creates one declared unsigned locked capture under the public session-content boundary, reports truthful depth/container facts, gives visible completion/error state, and returns to a responsive live surface. It starts no permission, App Attest, network, Photos, or queue work inside the extension. | `Pending` | `Pending` |
| 5 | Keep the extension live for the owner-approved soak duration and perform the approved capture/idle/interruption schedule. | Preview and controls remain responsive. Any real interruption or missing-frame event becomes an attributable visible recovering/unavailable state rather than a silent black/frozen UI, and follows the candidate's documented public recovery boundary. | `Pending` | `Pending` |
| 6 | Use the pre-approved system/user exit or suspension path. | The extension follows public lifecycle callbacks, releases or relinquishes camera/preview resources as designed, preserves successfully written session content for system migration, and does not use private dismissal or treat app opening as proof migration completed. | `Pending` | `Pending` |
| 7 | Open or resume the containing app only through the path approved for this candidate, without a handoff-time Library poll invented by the acceptance script. | Main camera/navigation remains interactive. The app does not block its first interactive surface while waiting for locked content, and import begins only when the public manager/update boundary exposes content. | `Pending` | `Pending` |
| 8 | Observe the public session-content exposure for the owner-approved window and inspect Pending Capture Queue/TAP Library. | Every valid locked capture is imported idempotently exactly once into the existing Pending Capture Queue and becomes visible as its pending/local TAP Library item. Import failure preserves diagnosable source content; normal later signing/Photos export remains outside the extension and outside this lifecycle pass condition. | `Pending` | `Pending` |
| 9 | Lock the device and relaunch the real Locked Camera control for every owner-approved repetition, including immediately after the prior suspend/import cycle. | Every repetition reaches root UI, a real first frame, and safe controls without first-tap freeze, black screen, stale prior session, duplicate controller/session ownership, or requiring an extra lifecycle round trip. | `Pending` | `Pending` |
| 10 | Capture again after relaunch, exit/suspend, and reopen the containing app. | The second and later cycles retain the same capture, migration, exactly-once import, and responsiveness properties as the first cycle. Previously imported records remain stable and are not duplicated or lost. | `Pending` | `Pending` |
| 11 | Terminate and relaunch the containing app through the approved normal path, then inspect the test records and launch Locked Camera once more. | Imported pending/local items persist once, normal app startup remains responsive, and the next locked launch again presents a real first frame without lifecycle regression. | `Pending` | `Pending` |
| 12 | **OWNER-LIVE:** review the synchronized lifecycle timeline and every recovery/import result. | The owner can correlate each physical action to one control, extension, content, import, and relaunch sequence and explicitly accepts or rejects the candidate. | `Pending` | `Pending` |

### Required Evidence

- Logs: one synchronized public-safe timeline for control intent, scene/root,
  session start/interruption/stop, first real frame, shutter/write, lifecycle
  transition, manager content update, import/duplicate decision, Pending Capture
  Queue visibility, resource release, and every relaunch.
- Screenshots/recording: continuous attended device recording covering lock-
  screen launch, first real frame, capture, complete soak, recovery if invoked,
  system-supported exit/suspend, main-app import visibility, and relaunches.
- Output artifacts: candidate public-API/dependency audit; declared unsigned
  session-content inventory; corresponding imported pending records; container,
  manifest/depth/proof-slot facts required by the candidate contract; and an
  exactly-once capture-to-record reconciliation table.
- Result bundles: branch/build/commit, entitlements, device/iOS, approved
  schedule and observation window, crash/spindump or symbolicated diagnostics
  for any stall, and focused automated tests used only as supporting evidence.
- **OWNER-LIVE:** written confirmation of first-frame visibility, control
  responsiveness, soak, capture, lifecycle exit, import, every relaunch, and
  the separation of historical E-series evidence from this result.

### Verdict Conditions

- **Pass:** the new `TAP-0013` candidate completes the owner-approved matrix,
  repetitions, soak, and observation window using public API only; every launch
  reaches a real first frame and safe controls; captures remain truthful;
  suspend/exit/relaunch never freezes or blacks out; content imports exactly
  once through the containing app; forbidden extension work is absent; and the
  owner accepts the result.
- **Fail:** any eligible run has an unexplained black/frozen/empty root, no real
  first frame, unsafe shutter, capture loss/corruption, private API, extension-
  side permission/App Attest/network/Photos/queue work, unresponsive system
  exit, next-launch freeze, dependency on an extra lifecycle round trip,
  duplicate/lost import, or no import within the approved observation window.
- **Blocked:** `TAP-0013` has no ready dedicated-branch candidate; only an old
  E-series build is available; the public API/dependency audit is incomplete;
  device/iOS, duration, repetition, exit path, or observation window is
  unapproved; the real lock-screen control/log chain is unavailable; the owner
  cannot attend; or diagnostics cannot distinguish candidate failure from an
  invalid test environment.

### Human Confirmation

`Pending — OWNER-LIVE attended execution and a separate promotion decision are required`

## Cold-path performance

This retains the measurement and regression requirements from the retired
TAP-0083 plan. The old plan is Expired, not completed; no performance verdict is
implied. Read this section only for requested cold-path work or device acceptance.
[ProductContract §2.7](ProductContract.md#27-startup-milestones-and-cold-path-execution)
owns the milestone and execution rules. Do not recreate a separate cold-path
standard or module README contracts.

Freeze one physical iPhone, source commit, scenario, installation/reset context,
S/P/I facts, Library/Pending counts, cache state, and evidence method. Use the
same artifacts where the configuration is unchanged:

| Configuration | Debugger attached | Launch detached from Home Screen |
| --- | --- | --- |
| Debug | Diagnostic comparison | Diagnostic comparison |
| Optimized Release/Profile, or equivalent TestFlight product | Diagnostic comparison | Required timing verdict |

For every cell:

1. Recreate the frozen installation, route, media scale, and cold/warm conditions.
2. Capture Δt0–t1 through Δt3–t4, namespaced S/P/C/L/V spans, deferred-work
   release, MainActor stalls, and the workload/owner associated with each span.
3. Exercise first Library entry, item open, and applicable cold resource/progress
   paths. A returned user action must publish its visible state before scalable
   work starts. Compare warm re-entry separately.
4. Check that equivalent Library snapshots do not republish; changed content or
   public errors do. Viewport changes do not invalidate the whole grid, and
   SwiftUI body does not repeatedly decode posters/thumbnails. Preserve existing
   prefetch behavior and cache invalidation/memory-warning handling when changing
   viewport, thumbnail, or cache work.
5. Burst progress independently of chunk count. Publications remain at most
   20 Hz while preserving the latest/terminal value, monotonicity, cancellation,
   and exact request identity. Presentation cleanup stays attempt-scoped.
6. Inspect actual route-shell/first-frame placement. Task.yield or session
   configuration alone is not proof that a visible frame committed.
7. Keep structured evidence public-safe and obtain the owner's run verdict.

Only an optimized detached run can close a measured timing threshold. No
threshold may be invented after seeing results; other cells diagnose compiler
and debugger amplification. Warm re-entry, a passing Web fixture, and deterministic
ordering tests cannot replace the cold physical run. Missing controlled inputs,
observable spans, or required device/owner participation means Blocked. Violated
ordering, stale publication, main-thread scalable work, or a failed frozen timing
budget means Fail. Record all four cells before a complete timing verdict.

The Video and Library sections also need the applicable cold checks before their
large-media/progress paths are accepted; their feature results are not a substitute
for this controlled startup matrix.
