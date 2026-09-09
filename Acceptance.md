# Acceptance Guide

Use this guide to check a specific TAPCam capability on a recorded build. It
separates observable behavior, artifact integrity, visual comparison, and
physical-device measurements. A successful build or simulated fixture is useful
evidence only for the boundary it exercises.

- [First-install operations](#first-install)
- [Camera readiness](#camera-readiness)
- [Professional controls](#professional-controls)
- [Field of view and depth alignment](#fov-depth)
- [Live Photo](#live-photo)
- [TAP Video](#video)
- [Production App Attest](#app-attest)
- [Library permissions and deletion](#library)
- [Visual parity](#visual-parity)
- [Locked Camera experiment](#locked-camera)
- [Cold-path performance](#cold-path)

## Prepare and record a run

Before the first action, record the source revision and installed build,
distribution and signing environment, device and OS, operator, installation/reset
method, selected cases, repetitions, stop conditions, and evidence location.
Declare performance and visual tolerances before measurement. Keep failed
repetitions in the result; a successful retry does not replace them. Report
selected, executed, passed, failed, and skipped counts, with reasons for omissions.
Do not generalize one tested device to an untested family.

Use designated disposable media. Delete-and-reinstall, system privacy resets,
asset deletion, and production backend operations require authorization for the
actual device, account, and targets. Do not erase unrelated Photos assets,
credentials, or pending captures. Record the exact reset: an overwrite install
may retain data and does not prove a fresh installation.

For physical behavior, an observer must attend the actions and record a textual
verdict. Collect JSON/JSONL logs and assertions before launch. Each trace needs
a schema version, run/scenario identifier, monotonic sequence and clock, source
commit/build/configuration, platform, activation type, S/P/I disposition, route,
event/checkpoint, workload, logical owner/executor, result, and correlation to
its triggering action. Record authorization and startup facts as public-safe
states. Public logs and reports must not contain media, user, device or credential
identifiers, private paths/URLs, location, assertion/proof
bodies, backend response bodies, or media bytes. Keep required originals, hashes,
redacted audit tables, and detailed diagnostics in controlled evidence storage.

**Pass** means every selected case meets its expected result with sufficient
evidence. **Fail** means an exercised requirement is violated. **Blocked** means
a required device, fixture, permission state, observation, or authorized operation
is unavailable; name it rather than substituting a simulated success. A partial
run establishes only its executed cases.

## Terms and evidence limits

**TAP Library** is the visible mixed-media grid and Viewer. The **Pending Capture
Queue** is private storage that serializes signing, Photos export/readback, retry,
and cleanup. A pending item has not yet completed that chain.

Startup routing uses three distinct facts:

- **S — Setup receipt:** completion for the current installation, locally bound
  to the credential established by the explicit first-install Network action.
- **P — Required permissions:** Camera authorized and Photos authorized or
  limited. Location, Microphone, and network health do not determine P.
- **I — Initialization marker:** atomic completion for the current app update,
  initialization schema, installation generation, and local device generation.

The route is missing/invalid S → Setup; valid S with unusable P → Required
Permission Check; valid S/P with absent or stale I → Resource Initialization;
valid current S/P/I → Viewfinder. A cold cache does not change this truth.

A local integrity pass means the complete original resource set matches its
embedded proof and content binding. It does not independently authenticate an
App Attest key or prove the scene, person, time, location, physical depth, non-AI
origin, non-recapture, or freshness. Backend signature verification is a separate
trust check. Zero-depth still photos and TAP Video remain eligible for signing
and export; depth availability is reported honestly and downstream depth
assessment is separate. Web and Simulator fixtures cannot establish physical
Camera/Photos, App Attest hardware, haptics, iCloud, thermal, or timing behavior.

<a id="first-install"></a>

## First-install operations

Use a physical device whose Camera, Photos, Location, and Microphone permissions
can be returned to `.notDetermined`, with controllable connectivity. Logs must
show setup actions, permission requests, initial App Attest attempts/timeouts,
PhotoKit observation/fetch/write, and camera warmup. Retain structured logs,
automated event-order assertions, and the observer's textual verdict only: no
photo, screenshot, or screen recording is acceptance proof for this section.

1. Delete and reinstall the recorded build. Confirm all four permission states
   are `.notDetermined`; otherwise the fresh-install case is Blocked. Start logs
   before the first process launch and leave Setup untouched for at least ten
   seconds. No prompt, Network attempt, observer, protected fetch/write, or
   camera warmup may start.
2. Background and foreground without tapping. Only passive status refresh is
   allowed.
3. While offline, tap Network once. One bounded initial App Attest sequence may
   retry at the configured intervals and must stop at its total timeout.
   Reachability or `/healthz` alone must not complete the row.
4. After timeout, background/foreground, wait at least one retry interval, and
   restore connectivity without tapping Retry. No new sequence may start.
   Tap Retry: exactly one new bounded sequence starts and completes only after
   initial App Attest registration/verification succeeds.
5. Tap each permission row's Allow action separately, resolving its prompt
   before proceeding. Every request belongs only to its corresponding tap.
6. In a separately reset round, complete Network, Camera, and Photos and Skip
   Location/Microphone. Skip must not prompt. Continue becomes enabled only
   after the three required operations complete; completing them alone does
   not leave Setup. Continue itself is exercised in camera readiness below.
7. In separate rounds, deny each permission in turn. Its row owns Open Settings
   recovery. Restricted states show stable guidance without promising a
   Settings action that system policy cannot provide.
8. After Setup, revoke Camera and then Photos. Launch or foreground: Required
   Permission Check contains only those required permissions and no Continue
   button. Recover from the affected row/system surface; targeted refresh
   automatically re-evaluates the route without starting Network, Location,
   or Microphone work or replaying Setup.
9. Correlate Photos logs: construction is observer-inert; no catalog scan occurs
   before the explicit eligible boundary; activation is idempotent with one
   observer; revocation/deactivation cancels queued refresh work.

Retain before/after authorization enums and an App Attest attempt table covering
explicit start, retries, timeout, lifecycle changes, network recovery, manual
Retry, and success. Assertions must also prove prohibited work stayed inactive.
Any implicit request, cross-row trigger, post-timeout restart, reachability-only
completion, or premature protected work is Fail. Unresettable permissions,
uncontrolled network, or insufficient hidden-work observability is Blocked.

<a id="camera-readiness"></a>

## Camera readiness

Use a physical camera/haptic-capable device after the required setup operations.
Provide a controlled readiness-delay/interruption method and inspection of S/I.
Record graph readiness, real preview presentation, safe shutter/controls,
haptics, first usable Library catalog, marker commit, Viewfinder entry, and
release of deferred work. Evidence is structured logs, assertions, and a live
textual verdict; retain no photo, screenshot, or screen recording as proof.

1. On a fresh install, complete initial App Attest, Camera, and Photos, leaving
   optional rows skipped if desired. Required completion alone must not enter
   the camera. Tap Continue and observe the separate Setup receipt write.
2. Resource Initialization remains visible until both groups complete:
   camera graph/path, a real first preview, safe shutter/primary controls and
   prepared haptics; plus the first usable Library identity/order snapshot.
   An empty successful snapshot qualifies. Originals, iCloud downloads,
   thumbnails, posters, hashes, ZIP, Share/controller prewarming, network, and
   queue completion must not be dependencies.
3. During readiness, shutter/control attempts cannot start unsafe work. Complete
   the camera group first and then the catalog group; repeat in reverse order.
   Neither alone may commit I or enter the interactive Viewfinder.
4. Inspect I as one atomic Application Support JSON record: bundle, version,
   build, initialization schema, installation generation, and local device
   generation match the run. Its commit follows all readiness conditions.
5. Immediately capture once and operate a FOV and primary mode/control. Confirm
   a real frame, expected haptic, responsive controls, and no black/stalled UI.
   Retain the capture-success event, not its image, as evidence here.
6. In a separate fresh-install round, terminate after Continue but before both
   readiness groups finish. Relaunch: S remains valid, I is absent/stale, and
   Resource Initialization returns without replaying Setup.
7. Delay each readiness group separately. The stable initialization page must
   offer no Failed, Retry, timeout, skip, or degraded entry. Bounded diagnostics
   identify the incomplete group. An inability to finish is an engineering
   defect, not a user recovery state.
8. With current S/P/I, start a new process on a Cold Resource Path. Setup and
   Resource Initialization are skipped. Revoke/restore Photos and then Camera:
   Required Permission Check returns automatically to initialization if I is
   stale, or Viewfinder if current.
9. Perform an in-place app update that retains S but invalidates I; launch
   offline. Initialization and camera interaction must not wait for later App
   Attest health/recovery or Pending Capture Queue work. Record overwrite
   development installs separately from this update case.

Verify local poster/recent-cover work and background credential/queue recovery
start only after the committed deferred-work release and their relevant guards.
A rejected diagnostic attestation is not itself a readiness failure; its
scheduling position is the relevant observation. Early marker commits, unsafe
controls, false preview readiness, credential-blocked entry, or permission
recovery that reopens Setup are Fail. Missing observability or safe fault
injection is Blocked. Measured launch timing uses the cold-path matrix below.

<a id="professional-controls"></a>

## Professional controls

Cover at least an eligible rear-LiDAR device and a non-eligible device. Record
capability, active device/format, generation, ISO, shutter duration, lens position,
exposure target offset, AF/MF state, clamp and risk state. Use stable light,
tripod, near/far focus targets, and a high-contrast exposure target; reset camera
preferences and exposure/focus values. Record control transitions and have an
observer judge brightness, focus, loupe behavior, haptics, and responsiveness.

`A/A`, `M/A`, `A/M`, and `M/M` describe automatic/manual ISO and shutter,
respectively. `S` on the camera control is shutter duration, unrelated to the
startup Setup receipt.

1. In Standard, exercise Basic EV and FOV without silently selecting the PRO
   LiDAR source. Enter PRO: transition frost remains until a real preview is
   interactive. Verify fixed rear LiDAR 24 mm / 1x, EV/ISO/S/Focus controls,
   read-only aperture ƒ, and no Standard Basic EV or FOV selector.
2. In A/A, move EV both ways and compare perceived brightness and readback.
   Exercise ISO midrange/limits in M/A and shutter midrange/limits in A/M,
   returning each to Auto. In M/M both values persist and EV is a read-only
   meter; restore one parameter at a time, then A/A.
3. Tap AF and drag temporary EV: its focus anchor stays put; a new tap resets
   temporary EV. Long press locks AF/AE; ordinary focus events do not clear the
   lock. An unlocked overlay clears on runtime invalidation, not a fixed timer.
4. Open Focus without dragging: AF remains. The first actual drag enters MF
   and locks current lens position before numeric writes. Continuous drags use
   latest intent without stale jumps.
5. In MF, tap near/far targets. Assist changes focus only, leaves exposure
   unchanged, and keeps shutter disabled until honest locked readback. Rapid
   taps and slider movement reject stale generations. The loupe centers the
   tap, main preview remains 1x, and disabling magnification preserves assist.
6. Switch rear PRO → front → rear. Frost lasts until destination interaction is
   safe; front exposes neither PRO nor MF; rear restores suspended intent
   without rewriting its saved preference. Exercise PHOTO/TAP VIDEO on the
   same eligible control path.
7. Exercise Default Off, Default On, Remember Last State, and safe Standard
   fallback. On the non-eligible device repeat startup/availability/front/rear:
   PRO stays absent while Standard works.

Retain the per-device EV/ISO/S/meter/lens table, capability/generation timeline,
recordings and relevant screenshots/artifacts. False capability, clamp/readback
mismatch, stale focus, exposure writes from assist, a second hardware preview,
primary-view crop, or early frost dismissal is Fail. Missing devices, targets,
or diagnostic readback makes that case Blocked.

<a id="fov-depth"></a>

## Field of view and depth alignment

Fix the camera on a tripod with stable light, center/edge landmarks, and near/far
objects that yield depth. Declare preview-to-output mapping and tolerance before
capture. Use default HEIC, reset Basic EV, suppress scene-changing flash behavior,
and record every FOV offered by the device. Keep the scene fixed throughout.

1. Record the default preview/FOV list. For each visible Standard FOV, select
   and stabilize it; record preview, RGB/depth sources, raw zoom and generation;
   capture; inspect original HEIC, manifest, crop/source facts, and Apple depth.
   Compare final RGB landmarks using the declared mapping, then RGB/depth
   landmarks in 2D.
2. Traverse all FOVs quickly in both directions and capture again. The final
   generation and explicit RGB/depth plan must own the artifact.
3. Enter PRO after readiness. The selector is absent and the eligible rear
   LiDAR path remains 24 mm / 1x. Capture at least three photos: fixed source,
   raw zoom 1.0, full frame without product crop, and matching RGB/depth facts.
4. Return to Standard. Its prior selection and mapping resume without PRO
   source or crop residue.

Retain per-FOV preview/original/manifest/depth comparisons, the three PRO sets,
generation logs, transition recording, and a human composition verdict. A visible
FOV without a capture plan, mismatch beyond the declared tolerance, stale plan,
or PRO crop/multi-focal output is Fail. A FOV not offered is unavailable, never
silently Passed. Missing usable originals, scene, depth, or tolerance is Blocked.

<a id="live-photo"></a>

## Live Photo

Live Photo binds one original primary HEIC/JPG and its original paired MOV.
Depth belongs to the primary photo; this procedure makes no per-frame MOV-depth
claim. Use physical devices supporting the path and a usable App Attest capture
credential. Observe a non-private moving scene and short audible cue.

Select devices and repetitions before capture. Cover these primary/audio states;
OS authorization and the app's data-use preference are independent:

| Primary format | Microphone OS permission | App audio data use | Expected audio |
| --- | --- | --- | --- |
| HEIC or JPG, tested separately | Granted | On | Captured cue |
| HEIC or JPG, tested separately | Granted or not granted | Off | No captured audio |

Record distribution/signing environment, permissions, free storage, and queue
count. Prepare exact-resource inspection before cleanup. Start public-safe logs
and attended capture/playback recording; exclude private speech from retention.

1. Check the selected format and both audio settings. An OS permission grant
   must not override an app-level opt-out. Press shutter once: one capture
   starts and the camera remains interactive.
2. Observe primary completion and the movie-complement callback. Success yields
   the selected primary container plus exactly one `paired-video.mov`. A failed
   complement may produce a valid still artifact, but never a partial artifact
   labeled/exported as Live Photo; it does not pass the Live Photo case.
3. Inventory and hash the exact originals before cleanup. No decoded frame,
   preview, adjusted Photos resource, or transcoded share output may replace
   them. Inspect manifest audio and primary-depth facts for truthfulness.
4. Let the private queue sign the complete pair. One capture assertion binds
   the exact canonical payload and resources without rewriting them. The final
   local gate validates the same signed primary/MOV pair before Photos commit.
5. Load original Photos `.photo` and `.pairedVideo` resources and reproduce
   their complete binding. Verification export must preserve bytes identical
   to readback; unsigned routing metadata contributes no trust.
6. Open the item in TAP Library, page away/back, then press and hold. The correct
   native Live Photo plays without blank substitution/crash and returns to the
   still presentation on release. Repeat after an iCloud download. Pinch and
   pan a RAW photo, then let its full-quality image load: zoom and viewed region
   remain stable. At minimum zoom, horizontal swipes page once per swipe;
   while zoomed, they pan the image. Cancel and complete a system edge-back
   gesture. Returning to the same item preserves the grid position; returning
   after paging reveals the current item.
7. Inspect the original MOV audio track and play it on an explicitly
   sound-enabled diagnostic/system surface. On rows reproduce the cue and
   report `audio = captured`; Off rows have no captured audio and report
   `audio = not-captured`. Record the Viewer's mute policy separately; captured
   audio does not require surprise sound during ordinary Library viewing.

Retain the settings matrix, resource inventory, event timeline, redacted proof
summary, exact pre-export/readback SHA-256 comparison, audio results, and human
motion/playback verdict. Lost/substituted resources, partial Live Photo export,
false audio/depth claims, skipped final validation, changed readback, or broken
native playback are Fail. Missing real device/credential, exact-resource access,
or controlled audio state is Blocked. Production credential trust is checked
separately below.

<a id="video"></a>

## TAP Video

Use real RGB and optional audio, observable finalized MP4 tracks, truthful
manifest/depth counters, and exact original-resource readback. Prepare sufficient
storage/power and record initial thermal state. Disable Low Power Mode for
comparable performance runs. Run audio Off and On independently; On requires
both OS permission and app data use. AirPlay, Picture in Picture, background
playback, and video 3D are outside this acceptance scope.

Declare device/iOS coverage, repetitions, CPU/RSS/dirty-memory/disk/thermal/drop
budgets, codec p50/p95 budgets, registration tolerance, and the method for proving
an original is iCloud-only. Use a reproducible zero-depth scene or controlled
hook that preserves real RGB/audio. Use a safe, documented writer-failure hook.

### Capture and lifecycle matrix

The following is a starting repetition budget; record any changes before the
run, not after its outcome:

| Journey for each eligible PRO device and audio state | Starting repetitions |
| --- | --- |
| New process → PRO → VIDEO → first recording | 10 |
| PRO Photo ↔ PRO Video | 20 |
| Standard VIDEO ↔ PRO VIDEO | 10 |
| Record five seconds and stop | 10 |
| AF taps, first MF tap assist after re-entry, full MF slider movement during recording | 10 each |
| VIDEO → front → rear PRO | 10 |

After every cycle check frost, recording state, Library entry, and next-recording
readiness. First-recording starvation, zero-duration RGB, black/frozen preview,
false recording state, or a missing prepared graph is Fail.

1. Inspect representative original MP4s, manifest/proof, RGB/audio/KLV tracks,
   coverage and drop counters. Recorded audio must agree with both settings and
   actual tracks; neither audio nor depth may be invented.
2. Confirm signed/exported/readback bytes bind correctly and the Library poster,
   foreground RAW playback, and registered 2D follow available facts.
3. Trigger writer failure. Recording UI exits promptly; the failed workspace
   does not enter the queue; the graph recovers and the next recording succeeds.
4. Produce valid RGB/optional audio with zero stored depth. Stop: one
   non-blocking Depth unavailable message appears; the original is retained,
   reports zero coverage, and proceeds through signing, Photos export and
   readback. RAW plays. Missing depth/registration may disable 2D but must not
   create a terminal capture or block export. Depth health is not integrity.
5. Record 15, 60 and 180 seconds at least three times per duration. Capture RSS,
   dirty memory, disk bytes, RGB/depth/audio drops, compression ratio, encode
   p50/p95 and thermal state. Gaps/drops remain visible in the manifest, memory
   does not grow linearly with duration, and all declared budgets hold.
6. On a real depth corpus, check exact-byte codec round trips and decode p50/p95.
   For a representative 180-second recording exercise RAW/2D, seek and dismiss
   with a symbolicated CPU trace. Repeat open → 2D → dismiss five times and
   inspect a memgraph. Trace file size is not heap evidence; whole-video `Data`
   or retained Viewer resources require investigation.
7. Capture known landmarks and measure same-frame registered-depth/RGB error
   against the predeclared display-space tolerance. Do not loosen it afterward.
8. Prove a signed original is not local, then open it. Keep poster/preview
   visible with one monotonic progress state. Swipe away/back and dismiss during
   download: underlying work cancels and stale callbacks cannot replace the
   current item. Reopen, finish, and confirm RAW/eligible 2D. Induce offline
   failure, restore connectivity, and exercise visible recovery.

Retain cycle tables, complete zero-depth state transitions, representative MP4
and readback audits, raw duration/codec/resource measurements, trace/memgraph,
registration results, real iCloud-only proof, and attended lifecycle recordings.
Apply the cold-path checks below to first-open/progress cases. Hidden media facts,
zero-depth terminal state, lost/corrupt artifacts, stale progress, unbounded
retention, or a failed declared budget is Fail. Missing real iCloud state,
corpus, observation, or measurement budget is Blocked for that case.

<a id="app-attest"></a>

## Production App Attest

Check four separate layers: the signed installed build's production environment;
backend acceptance of a credential; the device's offline assertion over the
canonical capture binding; and local final-byte validation before export and on
Photos readback. A backend `valid` response never replaces local file checks.
Diagnostic verification does not add a product Verify action on view or Share.

Use an authorized production account/install, request/capture budget, operator,
change window, stop conditions, retention and cleanup policy. A supported physical
device, inspectable Release/TestFlight app/archive, backend observability,
locally trusted verifier, exact-resource reader and disposable-copy mutation
harness are required. Choose fresh registration or reuse before running; do not
manufacture a fresh trace for a reuse case. Local `reset(photo_keyid)` removes
local metadata, not the Apple private key or backend credential.

Cover HEIC still, JPG still, Live Photo with selected primary/MOV pair, and TAP
Video with declared depth/audio states. Record repetitions and omissions. Archive
prior evidence; do not delete credentials, Keychain data, app data, or Photos
assets outside the named cleanup scope. Capture device/backend correlation
without secrets. The supporting [backend contract](BackendContract.md) and
[binding contract](bindings/capture-binding-and-proof-v1.md) define the local
wire requirements.

1. Inspect entitlements extracted from the exact installed signed `.app`, not
   source settings. `com.apple.developer.devicecheck.appattest-environment`
   must be `production`; app identity must match the backend. Runtime metadata,
   HTTPS deployment target and signed entitlement agree, with no development or
   localhost endpoint. The TAPCam production deployment is
   `https://www.tapnap.net`.
2. Run the selected preparation path. Fresh registration requires a backend
   challenge, atomic consumption, Apple attestation/app/environment/public-key/
   initial-counter validation, and backend acceptance before local persistence.
   Reuse requires an active matching production `photo_keyid` credential.
   Release reaches Ready after its health check without exposing backend/key
   details. The credential name is a lookup name, not identity or trust.
3. Capture each selected artifact and observe pending → signing → signed →
   exporting → original readback → exported, within budget and without unsigned
   fallback. Offline capture signing uses the canonical family binding and no
   online business-request assertion challenge.
4. Before Photos commit, validate exact HEIC/JPG, the complete Live Photo pair,
   or original MP4 as appropriate. Recompute binding and check expected identity
   and proof. Queue status and filenames cannot bypass the gate.
5. Read exact Photos originals and repeat local validation. Stream video to a
   disk-backed reader; do not substitute adjusted, compatibility, or re-encoded
   exports. In the diagnostic verifier, only after local reconstruction passes,
   submit `keyId`, `assertionObject`, and complete `signingBinding` to
   `/tapcam/capture-signatures/verify`. Do not upload media, manifest, depth, or
   a separate recomputed resource hash. The backend must recognize the active
   production key and return the contracted result.
6. View and open Share for completed owned captures. This must not initiate
   another backend Verify operation; protection/verifiability presentation
   remains distinct from diagnostic backend verification.
7. Alter disposable copies outside the proof slot: a still/Live primary, and
   a paired MOV or TAP Video. Every changed copy fails local validation before
   any backend call and cannot be exported as verified, even when the embedded
   assertion remains valid for the original binding.
8. Reconcile request counts, credential state, Photos assets and cleanup.
   Retain partial failures and redacted outcomes, never private media, full
   key IDs, assertions, proofs, raw responses or private URLs in public reports.

Evidence includes signed-product entitlements, runtime/backend environment,
credential-branch correlation, each artifact's queue/gate/resource inventory,
readback binding, local-before-backend order, mutation rejection, request counts
and cleanup. Environment mismatch, client-only trust, wrong/missing proof,
unsigned fallback, premature export, readback mismatch, media upload to the
signature endpoint, accepted mutation, or secret leakage is Fail. A development
credential, Simulator, test signer, or source inspection cannot substitute for
this production run.

<a id="library"></a>

## Library permissions and deletion

Use a disposable ordered Photos set with allowed/excluded Limited-access subsets,
a middle and terminal item, and a one-visible-item case. Prepare a genuinely
pending/local capture; do not relabel an exported asset or mutate its artifact.
Declare the large-Library item count and canonical sort order; no universal
minimum count or pixel scroll distance is implied. Record initial authorization,
install method, targets and source ownership before opening Library.

1. Grant Limited access through system UI. Open, leave and re-enter Library:
   only permitted Photos assets and eligible app-private items appear; no
   replacement permission prompt, expanded access or repeated request occurs.
2. Add/remove designated items in the system Limited selection and return.
   The grid refreshes without duplication, removed assets or replayed Setup.
3. Open an exported Photos asset, tap Delete and cancel. Exactly one system
   Photos confirmation appears, with no preceding app confirmation; cancel
   preserves asset, grid, Viewer and position.
4. Delete a middle exported item in a known order of at least three. Select
   the item formerly at the next index. Delete the terminal item: select the
   previous remaining item. Do not close Viewer while any item remains.
5. Open the genuine pending/local item and cancel its app-owned confirmation:
   no Photos deletion prompt appears and local record/artifact remain. Confirm
   deletion on the next attempt: remove only that local item through the queue
   storage boundary and use the same next-then-previous selection rule.
6. Delete the only visible item using the confirmation owned by its actual
   source. Viewer closes to empty Library without stale content.
7. In the large Library, scroll beyond the initial viewport, record a distinct
   allowed item, open it, page left/right when possible, return to that item and
   then the grid. The bookmark restores the clicked-item context at or near it
   without an unrelated top jump; exact pixel offset is not promised.
8. Leave Library completely and make a fresh entry. It starts at the top; the
   prior view-local offset is not durable across entries.
9. Repeat first entry with cleared caches at the recorded large catalog size.
   Check visible acknowledgement, semantic snapshot suppression, thumbnail
   decoding and bounded publication using the cold-path procedure below.

Retain public-safe authorization/snapshot/source/delete/selection/bookmark logs,
attended confirmation and return recordings, and controlled before/after target
inventories. Wrong confirmation ownership, destructive cancel, wrong-item
removal, excluded-content exposure, duplicate/stale sets, wrong adjacency, an
empty Viewer left open, or lost in-session context is Fail. Missing Limited
state, genuine pending item, disposable order or source-distinguishing evidence
is Blocked; Web/Simulator fixtures do not prove system deletion behavior.

<a id="visual-parity"></a>

## Visual parity

Compare an exact reference revision with a named native build, using a declared
state × media × iPhone viewport matrix. Record locale, Dynamic Type, Reduce
Motion, native icon semantics, tolerances and accepted platform differences
before comparison. Use read-only reference captures or live reference rendering;
do not change the baseline to hide a mismatch. State how each native state was
reached and distinguish deterministic fixtures from runtime observations.

Compare live side by side by default. Retain images/recordings only when the run
specifies that evidence form; otherwise use textual/JSON results, geometry/state
assertions and UI test bundles. A visual pass does not prove physical permissions,
camera, depth, signing, Photos, haptics, accessibility, performance or lifecycle.

| Surface or journey | Required comparison |
| --- | --- |
| First-install Setup | Row hierarchy, icon identity, explicit row actions, required/optional states, app/system boundary |
| Required Permission Check | Camera/Photos authorized, denied, restricted, Settings return and recovery; no Network, Continue, timeout, skip or fake system dialog |
| Resource Initialization | Preparing, camera-first, catalog-first and ready handoff; stable until both groups complete; no failure/Retry/timeout/degraded-entry or unrelated warmup UI |
| Viewer toolbar | System navigation Back; stable Photo/Live/Video media, pager/player and bottom chrome; Share/Delete vectors and circular backgrounds; compact centered RAW/2D/3D capsule |
| Share loading and credentials | Disabled until complete local/iCloud originals exist; local-check skeleton; Verified, Needs Retry and Failed states |
| Format selector | Native icons, copy, enabled/future states and media-specific resource requirements |
| Fast/slow preparation | Immediate thin determinate track only in the selected row's fixed subtitle slot; stable title/icon/badge/rows/popover/siblings |
| Failure, Retry and stale work | Public-safe failure in the same anchored surface; Retry only the failed option; old callbacks/artifacts cannot affect the new attempt |
| System handoff and dismissal | Popover disappears before one native activity controller; unchanged Viewer on return; attempt-scoped temporary cleanup |

Exercise Share on Photo, Live Photo and TAP Video without remounting the Viewer.
Opening the selector freezes complete originals and checks only local proof/
content binding; no backend/App Attest Verify request occurs. Photos assets
without a queue record can be Verified from embedded identity; Needs Retry is
queue-only. A local mismatch disables TAPNAP Package while direct Image/Video
retains an explicit unverifiability warning. Still/Live Package needs complete
valid resources; Video Package, Sticker and Link stay unavailable until their
respective formats exist.

For fast and slow preparation, other options remain visible but disabled.
Progress is monotonic, with no percentage, visible Cancel, delayed reveal,
artificial minimum hold, ready overlay or layout movement. The Share glyph stays
fixed with an additive ring. Payload readiness dismisses the selector immediately,
then constructs the system controller. There must be no blank frame, row flash,
full-screen loading replacement, toolbar rebuild, package prewarming or persistent
Share cache. Internal cancellation/stale cleanup does not add a public control.

Check localized copy, accessibility identifiers/values, Reduce Motion, materials,
spacing and optical platform differences. Use native controls and symbols rather
than fake system UI. Classify discrepancies as implementation defects, accepted
native variances, reference changes, or product conflicts; record the disposition
and human verdict for the exact reference/build pair. Missing reachable states
or an unsettled baseline/tolerance makes the affected cell Blocked. Hidden
mismatches, deprecated UI, altered baselines or fixtures claimed as runtime proof
are Fail. Physical-device evidence remains a separate result.

<a id="locked-camera"></a>

## Locked Camera experiment

Locked Camera is experimental; a successful lifecycle run does not establish
production support. Use a dedicated experimental build with a declared unsigned
capture layout, public-API/dependency inventory, and documented recovery behavior.
It must use SDK-public APIs only: no hidden selectors, private/programmatic
dismissal, dynamically linked `.tbd`-only transition symbols, or emulated private
lifecycle calls.

The extension owns locked UI, camera/depth capture and unsigned session-content
writes only. It must not request permissions, perform App Attest/network/Photos
export, capture Live Photo, or run the private queue. The containing app imports
content only when the public manager/update boundary exposes it; opening the
app is not proof that content migration completed.

Declare device/iOS, launch/exit paths, soak duration, capture/idle/interruption
schedule, repetitions, import observation window and safe recovery injections.
Install the recorded signed build, complete containing-app preparation before
locking, and inventory only designated test records. Use the real system Locked
Camera control; a direct debug view, preview or Simulator launch cannot replace
it. Start synchronized control-extension, capture-extension and app logs plus
attended recording before the first lock-screen action.

1. Inspect the binary/source API and dependency inventory. Forbidden extension
   work and private API must be absent.
2. Lock and launch through the system control. Observe control intent, secure
   scene, persistent root/controller, and visible non-empty UI. Camera
   configuration must be committed before starting the capture session.
3. Wait for a real first preview before shutter/controls become safe. Session
   configuration and placeholders do not qualify; black/frozen UI is failure.
4. Capture on the declared schedule. Each action writes one truthful unsigned
   resource to public session content, reports completion/error, and returns
   to responsive preview without forbidden extension work.
5. Run the complete soak and interruption schedule. Lost frames/interruption
   must enter an attributable recovery/unavailable state through public APIs,
   not silently freeze.
6. Exit/suspend through the system-supported user path. Release camera/preview
   ownership while preserving successfully written content for migration.
7. Open/resume the containing app without an invented Library polling step.
   Its first interaction stays responsive; import waits for public content
   exposure and enters the existing queue idempotently exactly once. Failures
   preserve diagnosable source content. Later signing/export is app-owned and
   separate from this locked-lifecycle result.
8. Within the declared observation window reconcile each capture with one
   pending/local Library item. Relaunch locked capture for every repetition,
   including immediately after suspend/import, and capture again. No stale
   session, duplicate controller, extra lifecycle round trip, lost content,
   first-tap freeze or black screen may appear.
9. Terminate/relaunch the containing app. Imported items persist once, normal
   startup stays responsive, and another locked launch reaches a real frame.

Retain the public API/dependency audit, exact unsigned layout and resource
inventory, synchronized action/session/frame/write/content/import/release
log, capture-to-record reconciliation, full soak/lifecycle recording and human
verdict. Capture loss, private APIs, forbidden extension work, unsafe controls,
failed exit/relaunch or missing import within the recorded window is Fail.
Unavailable real controls, build diagnostics, safe recovery, or declared
schedule is Blocked. Do not merge old experiment logs into a new run's evidence.

<a id="cold-path"></a>

## Cold-path performance

A **Foreground Process Launch** creates a process; **Foreground Resume** reuses
an existing scene. A **Cold Resource Path** has no usable relevant cache; a warm
path does. Record these independently from installation labels and S/P/I.
Measure one physical iPhone with the same source, scenario, route facts,
Library/Pending scale and cache-reset method in every cell:

| Build | Debugger attached | Detached launch from Home Screen |
| --- | --- | --- |
| Debug, same artifact in both cells | Diagnostic comparison | Diagnostic comparison |
| Optimized Release/Profile, or equivalent TestFlight product | Diagnostic comparison | Timing verdict |

Use these ordered milestones, with a monotonic clock:

| Milestone | Observation |
| --- | --- |
| t0 | Activation requested |
| t1 | First app-owned frame committed |
| t2 | Initial route surface stably committed |
| t3 | First real camera preview presented |
| t4 | Safe shutter/primary controls and haptics, plus any required first Library metadata snapshot; I commits here |
| t5 | Startup-critical interaction is protected and deferred work may release |
| tn | The selected journey reaches its terminal state |

For each of the four cells:

1. Recreate the recorded install, activation, route, catalog/queue scale and
   cold/warm conditions. Record Δt0–t1 through Δt3–t4, deferred release,
   MainActor stalls, and responsible work/executor. Label spans by startup,
   permissions, camera, Library and Viewer (S/P/C/L/V) without exposing IDs.
2. Before t1, permit only bounded local S/I and passive permission reads plus
   lightweight surface creation. Between t1 and t2, allow pure route reduction
   and bounded publication; camera/catalog work starts after its route surface
   commits. Background credential/queue recovery and offscreen media work wait
   for t5 and their guards.
3. Exercise first Library entry, item open, complete-original fetch/progress,
   and first system presentation where applicable. Publish visible feedback
   before scalable work. `async` is not evidence of non-MainActor execution:
   file I/O, hashing, ZIP, catalog enumeration, image decode, video copies and
   analysis need explicit isolation. A normal synchronous MainActor slice
   targets **under 8 ms**; scalable work cannot use that allowance as a loophole.
4. Equivalent Library snapshots do not republish; changed items/public errors
   do. Viewport changes must not invalidate the whole grid. SwiftUI body must
   not decode posters/thumbnails repeatedly. Preserve prefetch, cache
   invalidation and memory-warning behavior.
5. Generate a progress burst independently of chunk count. UI publication is
   **at most 20 Hz**, latest-value coalesced and monotonic, retaining terminal
   progress while rejecting cancelled/stale request identities.
6. Exercise success, cancellation, both SwiftUI/UIKit dismissal, presenter
   loss, construction/presentation failure and stale callbacks. Cleanup is
   idempotent and attempt-scoped; temporary resources survive exactly as long
   as the system consumer needs them, without retaining a persistent cache.
7. Check actual route-shell and first-frame commits. `Task.yield` or session
   configuration alone is not visible-frame evidence. Capture warm re-entry
   separately and retain the cold result and observer's verdict.

Only an optimized, debugger-detached physical run can support a measured timing
threshold. Record all four cells for a complete comparison; the other cells
diagnose compiler/debugger amplification. Do not invent a threshold after the
results or use a warm rerun, Web fixture, deterministic ordering test, Video run,
or Library run as a substitute for this controlled startup matrix. Missing
controlled inputs, observable spans or physical observation is Blocked. Violated
ordering, main-thread scalable work, stale publication or a failed declared
budget is Fail.
