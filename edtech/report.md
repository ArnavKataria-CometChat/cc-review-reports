# CometChat Skills Review — edtech

**Batch:** `2026-07-08-baseline` · **skills@** `ac87450a6be3c0f3300ac953942b5c8dd1b70c33` · **platforms:** backend, web, android, ios
**Repo:** ArnavKataria-CometChat/cc-review-edtech · **PR:** https://github.com/ArnavKataria-CometChat/cc-review-edtech/pull/1

## Headline

- Build pass: **3/4** · Trigger: **4/4** · Variant: **4/4**
- Feature completeness (avg): **87.5%** · Ease (avg): **3.5/5**
- Hallucinations: **0** · docs-escapes: **1** · retries: **9**
- Findings by tag: {'agent': 3, 'docs-mcp': 1, 'skills': 16, 'SDK': 7, 'harness-error': 3}

## Per-platform

| Platform | Build | Variant | Completeness | Ease |
|---|---|---|---|---|
| backend | ✅ | ✅ | 80% | 4 |
| web | ✅ | ✅ | 90% | 4 |
| android | ✅ | ✅ | 90% | 4 |
| ios | ❌ | ✅ | 90% | 2 |

## Skills exercised

`cometchat`, `cometchat-android-v5-calls`, `cometchat-android-v5-components`, `cometchat-android-v5-core`, `cometchat-android-v5-features`, `cometchat-android-v5-troubleshooting`, `cometchat-angular-calls`, `cometchat-ios-calls`, `cometchat-ios-components`, `cometchat-ios-core`

> **Reading this report:** every finding below is a skill / docs / SDK gap observed in the baseline integration. Where a fix was applied to the demo app during review, that is a *verification aid only* — the underlying gap remains **open** until the skill, docs, or SDK is updated. Fix status never downgrades a finding.


## Findings — gaps by owner (30 total)


### skills — the review's primary product  (16)

- (backend) E-ROLES — CometChat user provisioning fails on a fresh app: the integrated backend assigns role: student/tutor/parent/admin on POST /users, but custom roles must be CREATED (POST /roles) before any user can reference them — on a virgin CometChat app every user-provision call fails 'Failed to validate the data sent with the request' and the entire chat bootstrap is dead. Neither the skill nor docs mention role pre-creation; the integration agent wired RBAC roles it had no way to provision.
- (backend) E-GROUP-NOPROV — group sessions have NO CometChat group, so ALL group chat + group voice/video calling is dead (half the calling-spec matrix). Two layers: (1) the seed (api/management/commands/seed.py:245) creates group Sessions directly via Session.objects.create(kind='group',...) and NEVER calls cometchat.ensure_group — provisioning only runs in the API create/enroll views, which the seed bypasses; (2) SessionChatView returns the descriptor with group_guid='session-<id>' and can_call=true WITHOUT verifying the CometChat group exists. Verified: GET /v3/groups/session-1 -> 'group with guid session-1 does not exist', 0 members. Opening a seeded masterclass room loads a non-existent group -> group chat/call fails. All ensure_group/add_group_member calls are wrapped in cometchat.safe() (errors swallowed), so even API-created groups fail silently.
- (web) Calls ring but never connect (android/web) — CometChatCalls.init() is never called (WebRTC SDK uninitialized) and no ongoing-call session is wired (**/*.{ts,tsx,js,jsx,kt,swift}: required call-site `CometChatCalls\.init\(` not found)
- (web) E-WEB-CALLS — edtech Angular web app crashes at CometChat bootstrap (chat AND calling dead): 'TypeError: Cannot read properties of null (reading CallAppSettingsBuilder)' at cometchat.service.ts:74. CometChatUIKitCalls, re-exported from @cometchat/chat-uikit-angular, resolves to NULL at runtime under Angular's production esbuild bundle — the calls SDK is a UMD that the Angular UIKit re-export does not preserve through bundling. Because bootstrap throws before login, the ENTIRE chat surface never mounts (Gate 6 one_to_one_chat fails at step 0 open_chat). Angular analog of marketplace W6-ROOT (Vite). Skill/docs give no working Angular calls-init pattern.
- (web) E-NOCONV — a booked tutoring session does not provision a CometChat conversation: a student with 2 upcoming + 3 completed sessions sees 'No Conversations Yet' in Messages. The app models sessions in its own DB but never creates the corresponding CometChat 1:1 thread (or group for group-sessions), so the conversation only appears after a manual first message. Users expect their session's chat to be reachable from the session, not to have to cold-start a thread.
- (web) E-WEB-CALLCSS — web call UI renders unstyled/broken: the app imports only @cometchat/chat-uikit-angular/styles/css-variables.css (design tokens) but NOT the calls SDK's component stylesheet @cometchat/calls-sdk-javascript/dist/index.css, so the outgoing/ongoing/incoming call surfaces have no layout CSS (visible but broken). W9-class (unstyled kit) but specific to the calling components. Neither the skill nor docs list the calls index.css as a required import.
- (android) Calls ring but never connect (android/web) — CometChatCalls.init() is never called (WebRTC SDK uninitialized) and no ongoing-call session is wired (**/*.{ts,tsx,js,jsx,kt,swift}: required call-site `CometChatCalls\.init\(` not found)
- (android) Android chat has no call-initiate option — CometChatMessageHeader hides the voice/video call buttons by default (**/*.kt: required pattern `hideVoiceCallButton\s*=\s*false|hideVideoCallButton\s*=\s*false|CometChatCallButtons` absent)
- (android) Android app returns to HOME after a call ends — the UI Kit's ongoing-call activity finishes in its own task and nothing re-foregrounds the app (**/*.kt: required pattern `FLAG_ACTIVITY_REORDER_TO_FRONT` absent)
- (android) Android ghost call — the UI Kit's ongoing-call activity is not finished when the remote party ends a 1:1 call; onCallEndedMessageReceived is a default no-op the app never overrides, so the local user is stuck in a conference-of-one until they manually hang up (**/*.kt: required pattern `onCallEndedMessageReceived` absent)
- (android) Android ghost call — the UI Kit's ongoing-call activity is not finished when the remote party ends a 1:1 call; onCallEndedMessageReceived is a default no-op the app never overrides, so the local user is stuck in a conference-of-one until they manually hang up (Maestro assertion failed [calling.yaml] on android: 
Waiting for flows to complete...
[Failed] edtech-android-calling-smoke (33s) (Assertion is false: "Chat.*|Messages" is visible)

1/1 Flow Failed


)
- (android) Android incoming-call overlay covers the conversation header — CometChatIncomingCall (UIKit v6) mounted as an always-on overlay renders its Accept/Decline chrome with no bound call, hiding the header (peer name, back, voice/video call buttons) so a call can never be started (Maestro assertion failed [calling.yaml] on android: 
Waiting for flows to complete...
[Failed] edtech-android-calling-smoke (33s) (Assertion is false: "Chat.*|Messages" is visible)

1/1 Flow Failed


)
- (android) Android call buttons untappable — the conversation screen renders edge-to-edge with the CometChat message header UNDER the status bar; the voice/video call buttons land in the status-bar touch region, so they are visible but untappable and a call can never be initiated (Maestro assertion failed [calling.yaml] on android: 
Waiting for flows to complete...
[Failed] edtech-android-calling-smoke (33s) (Assertion is false: "Chat.*|Messages" is visible)

1/1 Flow Failed


)
- (android) RBAC 'parent reads own child's threads' / 'admin audits' is not actually implemented — parents and admins get a static 'Read-only observer' notice instead of the child's/session's messages. Client-only rendering of the logged-in user's own conversations makes faithful cross-user observation impossible without server-side conversation membership, but the requested read leg is therefore unmet on the client.
- (android) E-AND-GHOST — android calling wedges after the first call (second call never connects): CometChatManager registers call listeners but NEVER releases the call session — no CometChat.endCall(sessionId) and no clearActiveCall at connect/login or on an incoming ring. After any abnormal end (backing out, app kill, missed) the user-pair stays BUSY server-side and the next call auto-rejects. Same session-lifecycle gap as marketplace W11/A11/I12 and iOS I4.
- (ios) CometChatCallEventListener extension defines several on*-prefixed methods (onCallEnded, onIncomingCallAccepted, onOutgoingCallAccepted, onIncomingCallRejected, onOutgoingCallRejected, onCallInitiated) that are not requirements of that UI-Kit protocol (which uses cc*-prefixed events); at best dead no-ops, at worst evidence the agent guessed the event surface. The intended teardown-on-remote-end signal is ambiguous.

### docs-mcp — documentation coverage  (1)

- (backend) The idempotent 'already exists' handling depends on error-code strings the agent could not confirm from docs (ERR_UID_ALREADY_EXISTS, ERR_ALREADY_JOINED). If the real codes differ, repeat provisioning/enroll will raise CometChatError instead of being treated as success (fails loudly rather than silently, but breaks the intended idempotency).

### SDK — CometChat SDK packaging / API  (7)

- (web) Parent/admin read-only observation is not actually implemented: the UI Kit only renders the logged-in user's own conversations, so a parent 'reading own child's threads' or an admin auditing a session sees a static read-only placeholder note (observerReason()) instead of the real thread. The use-case RBAC requirement (parent reads own child's threads; admin audits) is therefore only partially met — the send/call gate is enforced, but the actual observation surface is a stub.
- (android) Android chat shows no calling option — CometChatMessageHeader (auto call buttons) / call listener / incoming-call overlay absent (RELATED to X1 app-calling root cause) (**/*.kt: required call-site `CometChatMessageHeader` not found)
- (android) Android chat shows no calling option — CometChatMessageHeader (auto call buttons) / call listener / incoming-call overlay absent (RELATED to X1 app-calling root cause) (Maestro assertion failed [chat.yaml] on android: 
Waiting for flows to complete...
[Failed] edtech-android-chat-smoke (31s) (Assertion is false: "Chat.*|Messages" is visible)

1/1 Flow Failed


)
- (android) E-AND-ROOM — integrated Android session room CRASHES on entry (chat/calling unreachable): tapping 'Enter Session Room' opens SessionRoomActivity which force-finishes with NullPointerException at com.cometchat.chatuikit.messagelist.MessageAdapter.<init>(MessageAdapter.java:518), via CometChatMessageList.initViewComponent (CometChatMessageList.java:559) during the view's constructor. The UIKit MessageList inflates before its required config/SDK state is available (A7-family: SDK/UIKit not fully initialized in this activity's context before the <cometchat-message-list> view is constructed). App returns to My Sessions; the entire CometChat surface (chat + voice/video call buttons) is unreachable on Android. Reproduced live 2026-07-16 on the integrated build.
- (ios) CometChatSDK 4.1.6 CocoaPods release omits Vendors/CometChatStarscream.xcframework even though its podspec lists it under vendored_frameworks; the binary has undefined CometChatStarscream WebSocket symbols and an @rpath load command, so a CocoaPods-only install links but crashes at launch with 'Library not loaded: @rpath/CometChatStarscream.framework'. Genuine upstream packaging defect. RE-SCOPED 2026-07-16: the fix does NOT require SPM — a local CocoaPods podspec sourcing the same real CometChatStarscream_1_0_2.xcframework from CometChat's CDN resolves it (plus `pod 'CometChatCardsSwift', '~> 1.1'` for the UIKit companion). Verified building+linking+embedding on edtech 2026-07-16.
- (ios) I7 reproduced on edtech (registry I7); RE-SCOPED 2026-07-16. Real gap: CometChatSDK 4.1.x hard-imports CometChatStarscream and CometChatUIKitSwift 5.1.16 hard-imports CometChatCardsSwift, but the published CocoaPods artifacts don't bundle/declare these companions, so a by-the-book `pod install` won't build/link ('No such module', 'Symbol not found: WebSocketEvent'). The earlier framing ('statically linked into CometChatSDK; builds clean but crashes at launch') was WRONG and led to a non-working dSYM/stub hack. Known app-side workaround (same as telehealth + marketplace): add `pod 'CometChatStarscream', :podspec => 'CometChatStarscream.podspec'` (local podspec -> real xcframework from CometChat's CDN) + `pod 'CometChatCardsSwift', '~> 1.1'`. VERIFIED 2026-07-16: BUILD SUCCEEDED on the iOS Simulator; LiveTutor.app embeds+links CometChatStarscream/SDK/UIKit/Calls/WebRTC. Severity re-scoped blocker -> known-workaround; finding stays recorded (the pod/docs packaging gap is real).
- (ios) iOS GROUP chat crashes on entry (masterclass unreachable), newly discoverable only after the I7 build fix. Opening a group session's CometChat conversation that contains call-event bubbles hard-crashes on the main thread INSIDE CometChat's own UI Kit: CometChatCallBubble.setupStyle() at CometChatCallBubble.swift:164 (protocol witness for RawRepresentable.rawValue.getter in conformance CallType / specialized == infix), Exception 'Instruction Abort - Translation fault', pc=0, via UITableView cell display -> CometChatCallBubble.willMove(toWindow:). ISOLATION: a 1:1 conversation containing call-event bubbles (Call ended / Outgoing Call / Call accepted) renders fine and does NOT crash — so iOS 1:1 chat works; the crash is specific to the GROUP conversation's call bubble (group/meeting CallType). Verified live 2026-07-16 on the integrated iOS build (Simulator, iPhone 17 Pro) after fixing I7 (CometChatStarscream + CometChatCardsSwift pods). Crash is entirely in CometChatUIKitSwift compiled code, not app code — not app-fixable without hiding call history. Blocks the iOS group-call test.

### agent — integrating-agent behaviour  (3)

- (backend) The RBAC scope 'parent reads own child's threads' and 'admin audits' is not actually implemented — the backend only returns can_send/can_call=false flags plus conversation identifiers, and relies on the client honoring them. Parents/admins are never added to the 1:1/group conversations and no read proxy exists, so there is no real mechanism by which they can read the threads. The read-only audit half of the RBAC requirement is effectively unimplemented.
- (web) The Messages inbox relies on an undocumented/unverified auto-wiring: <cometchat-conversations> with no (onItemClick)/(itemClick) handler is assumed to set active conversation state read back via ChatStateService.activeUser()/activeGroup(). If ChatStateService does not auto-populate active conversation on click (no explicit binding is present), clicking a conversation never opens a thread and the inbox thread pane stays empty. This integration path is not exercised by the auto-caught W-issues.
- (ios) Build fails (build_pass=false) after 3 retries. Despite the local CometChatStarscream.podspec workaround, the app still does not build, and the committed `scripts/reconstruct-cometchat-companion-modules.sh` that supposedly 'reconstructs CometChat companion modules from Pods dSYMs' into vendor/ is not a viable mechanism (Swift modules cannot be reconstructed from dSYMs) — the agent shipped a non-working, non-standard build hack rather than a verified-green integration. [RESOLVED 2026-07-16 by orchestrator: replaced the non-working dSYM-reconstruction hack with the correct workaround — a local CometChatStarscream.podspec sourcing the real xcframework from CometChat's CDN + `pod 'CometChatCardsSwift', '~> 1.1'`; BUILD SUCCEEDED. The agent lapse (shipping a non-standard hack instead of the known pod fix) stays recorded.]

### harness-error  (3)

- (web) G6-FORKBOMB — Gate 6 heal layer recursed into a process fork bomb (observed: 764 gate6.run processes). Chain: a failed choreography triggers _heal_layer (gate6/run.py:202) which spawns a diagnose-only debug agent; the agent's dossier instructs 'reproduce by running the repro command' = 'python3 -m gate6.run <chor>'; that re-invocation fails, triggers ITS OWN heal layer, spawns ANOTHER agent, ad infinitum. spawn_cap_per_stage did NOT bound it because the cap only limits one in-process debug_clusters call — the recursion is across separate OS processes, each with a fresh cap. Fixed: CC_GATE6_NO_HEAL env guard (a debug-agent-launched gate6.run runs the choreography for evidence but never re-heals), propagated from lib/debug_agent to any child gate6.run; self_test fixture added.
- (web) G6-KITDOM — the Gate 6 web executor (gate6/executors/web_driver.mjs) was written against the REACT CometChat kit (marketplace) and does not cleanly drive the ANGULAR kit (edtech). Concrete DOM differences found live: nav label 'Messages' vs 'Chat'; conversation rows are .cometchat-paginated-list__item / .cometchat-conversation-item (Angular) vs .cometchat-list-item (React); the <cometchat-message-header> host is display:contents (no box, never 'visible' to Playwright) vs a real visible div. Partial cross-kit fixes applied (nav param, dynamic row selector, attached+composer header check), but per-app row-open still needs tuning. This is a HARNESS limitation, NOT an app finding — recorded harness-error so it never contaminates the skills ledger.
- (android) E-AND-CALLUI (A12-class) — android in-call screen is blank gray/black on first mount; call connects (audio live), back-out+reopen paints it. CONFIRMED emulator GLES->Metal compositing artifact, not an app bug: ro.hardware.egl=emulation and SurfaceFlinger renderer='Android Emulator OpenGL ES Translator (Apple M4), OpenGL ES 3.0 (4.1 Metal)'. The CometChat ongoing-call SurfaceView (WebRTC video) sits on a separate hardware layer the emulator's GLES->Metal translator fails to present until a window reconfiguration forces recomposite (why back+reopen fixes it; plain rotation sometimes doesn't recreate the SurfaceView layer). Two app-side mitigations already FAILED on marketplace (SurfaceView recreate = no-op; visibility toggle = fully black, reverted). Will NOT reproduce on a physical device. Definitive test: real device OR emulator relaunched with -gpu swiftshader_indirect (software SurfaceView).

## Prioritized improvement suggestions

- **[docs-mcp]** (backend) On the /rest-api/users/create doc page, add the conflict/'already exists' error code and an explicit idempotent create-or-update (POST → PUT on conflict) pattern, matching what the Groups error table already documents.
- **[docs-mcp]** (backend) Reconcile the UID character-constraint wording between /rest-api/users/create ('alphanumeric with dashes only') and the SDK Key Concepts pages ('alphanumeric, underscore, hyphen') so a single canonical rule is documented.
- **[docs-mcp]** (backend) Add a docs-mcp guide on observer / read-only access patterns (e.g. how a parent or admin actually reads another user's conversations — group membership with a restricted scope, or a server-side history proxy via REST), and clarify that a `role` string on a CometChat user does not itself enforce messaging or grant read permissions.
- **[skills]** (backend) The backend skill must create the app's roles (idempotent POST /roles) before assigning them — or document that role assignment requires prior role creation; every RBAC-enabled use case hits this on a fresh app
- **[skills]** (backend) Lazily ensure the CometChat group exists when the chat descriptor is requested (idempotent ensure_group + add current member in SessionChatView), so groups work regardless of how the session was created (seed, API, migration). And do not swallow provisioning errors silently — surface them so a broken group is visible
- **[docs-mcp]** (web) Add a docs-mcp section on cross-user thread observation/audit patterns, clarifying that the client UI Kit can only render the logged-in user's own conversations and that read-only oversight of another user's/session's thread requires server-side moderation/audit APIs or explicit group membership. Include the recommended approach for 'parent reads child's threads' and 'admin audits' RBAC roles.
- **[skills]** (web) Document the canonical way to drive conversation selection in the Angular UI Kit (e.g. bind (onItemClick) on <cometchat-conversations> and set the active user/group explicitly) rather than relying on implicit ChatStateService auto-wiring, so the Messages inbox thread pane reliably opens on click.
- **[skills]** (web) The Angular skill must import CallAppSettingsBuilder from the calls SDK directly (@cometchat/calls-sdk-javascript) rather than the null UIKit re-export, and document the bundler-safe calls-init for Angular — same gap as the Vite/React path (W6-ROOT)
- **[skills]** (web) On session booking (and for existing sessions on first load), the backend/skill should provision the CometChat conversation — create the group for group-sessions and ensure the 1:1 thread exists — so Messages reflects the user's actual sessions
- **[skills]** (web) The web skill must import @cometchat/calls-sdk-javascript/dist/index.css (angular.json styles / global import) alongside the chat kit css-variables — the calling components ship their layout CSS separately and it is not auto-injected
- **[skills]** (android) Add a 'read-only observer / third-party conversation viewing' recipe to the Android v5 calls/components skill (and its web equivalents): document that the client SDK only surfaces the logged-in user's conversations and give the sanctioned pattern (backend adds the observer UID to the group/1:1 as a muted member, or an audit REST feed) so agents don't silently drop the parent/admin read leg.
- **[skills]** (android) Correct the cometchat-angular-calls skill (§1.0 + anti-pattern 9): stop labeling an explicit CometChatCalls.init a 'double-init' anti-pattern; state that setCallingEnabled(true) does NOT initialize the standalone Calls/WebRTC SDK and that an explicit CometChatCalls.init(callAppSettings) is required, mirroring the Android calls skill.
- **[skills]** (android) Document in the Android v5 components/troubleshooting skill the exact CometChatMessageHeader API for showing/hiding the voice/video call buttons (both the hideVoiceCallButton/hideVideoCallButton constructor/XML flags and the runtime setVoiceCallButtonVisibility/setVideoCallButtonVisibility setters), since buttons are hidden by default.
- **[skills]** (android) Ensure CometChatUIKit.init completes (and the active theme/config are set) BEFORE inflating CometChatMessageList — gate the SessionRoomActivity's kit views on init like the app does elsewhere, or the skill must document that the message-list view must not be constructed before UIKit init resolves
- **[skills]** (android) The android skill must wire call-session hygiene: clear/end a stale active call at connect and on an incoming ring for a different session, and endCall on abnormal teardown — the SDK provides no default cleanup and every integrator hits the 'first call works, second doesn't' wedge
- **[skills]** (ios) Add a cometchat-ios-core (or ios-calls) gotcha documenting that the CocoaPods CometChatSDK 4.1.x release ships without CometChatStarscream.xcframework (referenced by its own podspec) and that the fix is a local podspec sourcing https://library.cometchat.io/ios/v4.0/xcode15/CometChatStarscream_1_0_2.xcframework.zip — include the ready-to-paste podspec so agents don't invent dSYM-reconstruction hacks.
- **[docs-mcp]** (ios) Add a docs-mcp entry indexing the CometChatStarscream packaging gap and the exact SPM binary download URL so the query resolves in-tool instead of forcing a web escape to raw.githubusercontent.com Package.swift.
- **[skills]** (ios) In cometchat-ios-calls, document the exact CometChatCallEventListener protocol surface (cc*-prefixed methods: ccOutgoingCall/ccCallAccepted/ccCallRejected/ccCallEnded) and the correct SDK-level signal for remote 1:1 call end plus how to dismiss CometChatOngoingCall, so agents stop adding non-protocol on* methods and hand-rolling window sweeps.
- **[skills]** (ios) CometChatCallBubble.setupStyle() must handle every CallType it can receive in a GROUP conversation without dereferencing a null/mismatched RawRepresentable witness. Likely a version-skew or unhandled group/meeting CallType between CometChatUIKitSwift 5.1.16, CometChatCallsSDK 5.0.1 and CometChatSDK 4.1.6; ship a CallBubble that degrades gracefully for unknown/group call types instead of crashing.

## docs-mcp — infra note (coverage UNMEASURED this run)

- (backend) Create-user REST page does not document the 'already exists' error code (agent had to guess ERR_UID_ALREADY_EXISTS from the Groups error table) — no idempotent re-provisioning / conflict-handling guidance for POST /users.
- (backend) No backend/server-side auth-token-minting recipe surfaced for a Python/Django framework specifically; agent stitched it together from individual REST endpoint pages (worked, but no end-to-end backend guide).
- (backend) Docs do not explain how a non-participant (parent/admin) actually gains read-only access to another user's 1:1 or group conversations — setting a `role` string on a CometChat user does not by itself enforce or grant read access.
- (android) No doc/skill pattern for read-only / third-party conversation observation (parent reads own child's threads, admin audits) — the client SDK only renders the logged-in user's own conversations, so the agent had to punt this RBAC leg to a static 'read-only observer' notice.
- (android) One docs-mcp query was empty text (returned nothing); likely an agent-side artifact rather than a docs coverage gap, but it produced no signal.
- (ios) No docs-mcp coverage for the CometChatSDK 4.1.6 CocoaPods packaging gap (missing CometChatStarscream.xcframework) or the SPM binary download URL (library.cometchat.io/ios/v4.0/xcode15/CometChatStarscream_1_0_2.xcframework.zip) — the agent had to web-escape to raw.githubusercontent.com Package.swift to discover it.
- (ios) No docs-mcp query is recorded for the Django/python backend auth-token flow (/cometchat/token minting via REST API key); backend auth guidance for python was expected from docs-mcp and is not evidenced.
- (ios) The single non-empty query about dismissing the ongoing-call UI did not yield a first-class API — the agent fell back to manually sweeping every scene window to dismiss CometChatOngoingCall, suggesting docs did not clearly answer how the ongoing-call screen dismisses on remote 1:1 end.