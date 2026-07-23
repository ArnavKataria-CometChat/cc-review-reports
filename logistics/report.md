# CometChat Skills Review — logistics

**Batch:** `2026-07-08-baseline` · **skills@** `ac87450a6be3c0f3300ac953942b5c8dd1b70c33` · **platforms:** backend, web, android-ios
**Repo:** ArnavKataria-CometChat/cc-review-logistics

## Headline

- Build pass: **3/3** · Trigger: **2/3** · Variant: **2/3**
- Feature completeness (avg): **90.0%** · Ease (avg): **4.0/5**
- Hallucinations: **0** · docs-escapes: **0** · retries: **7**
- Findings by tag: {'agent': 3, 'skills': 2, 'harness-error': 4, 'docs-mcp': 2, 'SDK': 1}

## Per-platform

| Platform | Build | Variant | Completeness | Ease |
|---|---|---|---|---|
| backend | ✅ | ✅ | 80% | 4 |
| web | ✅ | ❌ | 100% | 4 |
| android-ios | ✅ | ✅ | 90% | 4 |

## Skills exercised

`cometchat`, `cometchat-flutter-v5-calls`, `cometchat-flutter-v5-conversations`, `cometchat-flutter-v5-core`, `cometchat-flutter-v5-events`, `cometchat-flutter-v5-messages`, `cometchat-production`

> **Reading this report:** every finding below is a skill / docs / SDK gap observed in the baseline integration. Where a fix was applied to the demo app during review, that is a *verification aid only* — the underlying gap remains **open** until the skill, docs, or SDK is updated. Fix status never downgrades a finding.


## Findings — gaps by owner (12 total)


### skills — the review's primary product  (2)

- (backend) No server-side Java example exists in the production skill for the auth-token/user/group REST flow, forcing a full manual translation to java.net.http for a legacy JSP/servlet stack that is an explicit target here.
- (android-ios) iOS ghost call — the UI Kit's ongoing-call screen is not dismissed when the remote party ends a 1:1 call; the local user is stuck in a conference-of-one with the timer running until they manually hang up ([ios] Maestro assertion failed [calling.yaml] on ios: 
Waiting for flows to complete...
[Failed] logistics-ios-calling-smoke (1s)

1/1 Flow Failed


)

### docs-mcp — documentation coverage  (2)

- (web) docs-MCP react-uikit-quickstart bundle is stale for v6: it still shows the removed CometChatConversationsWithMessages component, forcing the agent to discover the composed Conversations + MessageHeader/List/Composer shape from the skill file instead of docs.
- (web) No calling bundle in docs-MCP: init order (separate CometChatCalls.init after CometChatUIKit.init), incoming-call mounting, and callee-side teardown all had to come from the cometchat-react-calls skill; docs-MCP only covered the call-events symbol page.

### SDK — CometChat SDK packaging / API  (1)

- (web) Calls SDK types `region` as a narrow union ('us'|'eu'|'in') while the backend supplies a runtime-validated string, forcing a type assertion at CometChatCalls.init — a minor SDK ergonomics gap that pushes an unsafe cast into integration code.

### agent — integrating-agent behaviour  (3)

- (backend) The RBAC isolation rule ('drivers and clients NEVER converse directly; dispatcher is the hub') is asserted but not actually enforced. The backend only avoids putting them in a shared group — CometChat 1:1 messaging is open by default, so any provisioned driver could still DM any provisioned client. No blocking/restriction mechanism is applied to back the hub claim.
- (android-ios) 1:1 direct thread/call opens the counterparty with the wrong display name: route_detail_screen.dart calls _openDirect(context, directUid!, directLabel) and constructs cc.User(uid: uid, name: name) where `name` is the UI label ('Message driver (1:1)' / 'Message dispatcher (1:1)'), not the person's real name. The MessagesScreen header and the outgoing call title therefore literally read 'Message driver (1:1)' instead of the driver/dispatcher name.
- (android-ios) Route/exception threads are entered by pushing client-fabricated stub cc.Group(guid:'route-<id>'/'exception-<id>', type:'private') and cc.User(uid:...) objects straight into CometChatMessageList / call APIs without fetching or joining the target first. Correct behavior depends entirely on the backend having provisioned membership under the mirrored id scheme; unlike ChatsInboxScreen (which has an EmptyState fallback), these screens have no user-facing failure path if the group is unprovisioned or the join fails.

### harness-error  (4)

- (web) 0 screenshots captured — web flow driver lacks captureShot (update the use case's _driver.mjs) (/Users/admin/Desktop/orchestrator/cometchat-skills-reviewer/cometchat-skills-reviewer/_reports/2026-07-08-baseline/logistics/web/smoke)
- (android-ios) ios smoke drove the pre-installed app — the built sim app is --no-codesign (I9: keychain -34018 breaks CometChat login), so it is NOT auto-installed and freshness is UNVERIFIED ([ios] /Users/admin/Desktop/Allapplications/logistics/mobile/build/ios/iphonesimulator/Runner.app)
- (android-ios) HARNESS BUG (not app/SDK/skills): the logistics Gate 3 mobile smoke flows are unimplemented stubs. smoke/flows/logistics/{android,ios}/{chat,calling}.yaml each contain only `- launchApp: {clearState: true}` plus a comment ('dispatcher signs in via the demo card... exercises calling') that was never turned into Maestro steps — byte-identical across both platforms and all three captured runs (baseline + flake_rerun_1 + flake_rerun_2). Because clearState wipes the app to the login screen and the flow ends in ~2s ([Passed] in calling.yaml.out.log; junit status=SUCCESS) without signing in or navigating, _capture_mobile_screenshot grabs the login screen and the screenshot judge reports 'Screen shows a login form instead of any call UI'. The fingerprint visual/screen-shows-a-login-form-instead-of-any-call-ui superficially resembles an app calling defect (A-series/X1) but the app was never exercised: the Flutter app has full calling wiring (cometchat_service.dart CometChatUIKit.init+CometChatCalls.init+loginWithAuthToken; call_screen.dart joinSession; call_manager.dart). Compare smoke/flows/delivery/android/calling.yaml, a complete live-verified flow, to see what the logistics flow was supposed to be. Two harness defects: (1) the logistics smoke flows were left as launch-only stubs; (2) the pre-integrate linter lint_calling_spec.py only checks Gate 3 flow-file EXISTENCE (check 2) and applies its [NO-STUB-COVERAGE] anti-stub guard only to Gate 6 choreographies (check 4), so the stub sailed through. RECOMMENDED HARNESS FIX: (a) author the logistics chat/calling flows with real login+navigation+call steps modeled on the delivery flow; (b) extend the [NO-STUB-COVERAGE] check (or add a flow-content lint / a classify_harness_error rule) so any smoke flow whose only command is launchApp is failed as a harness_bug rather than passed and visually judged. This keeps the failure OUT of the app/skills gap ledger — it measures nothing about the skills under review.
- (android-ios) MISATTRIBUTION (harness bug): The gate-3 visual failure 'Screen shows a route list, not a chat conversation' on logistics android-ios is caused by an UNAUTHORED Maestro smoke flow, not by the app. chat.yaml (and calling.yaml) contain only `- launchApp: {clearState: true}` plus a prose comment describing the intended steps ('opens the Chats tab, exercises chat') — no tapOn/assertVisible/input exists (flows byte-identical across smoke, flake_rerun_1, flake_rerun_2). The flow never navigates to chat, so it screenshots whatever screen the app happens to show after launch; the same stub yields login (today's android shots), 'Route Board' (dossier + rerun_1 ios), and a blank frame (rerun_1 android calling) across runs — dispositive evidence the flow, not the app, controls the screen. The app is correctly integrated: home_shell.dart:33-39,76 places a one-tap 'Chats' tab on every role opening ChatsInboxScreen (chats_inbox_screen.dart:33) which renders the real CometChatConversations UI Kit over a properly-initialized CometChatService (cometchat_service.dart:44-60, including the X1 CometChatCalls.init split). Remediation is harness-side: author the flow's navigation (tapOn demo 'Dispatcher' card -> tapOn 'Chats' tab -> tapOn a conversation -> assertVisible message composer/bubbles) before judging; do NOT file this as an app/skills/SDK chat defect. This is the same empty-flow-read-as-app-failure class as the prior 'ghost call' misattribution called out in the debug contract.

## Retracted, demoted & methodology notes (4)

_Removed from the counted findings because they were false positives, duplicates of a kept finding, harness-infra artifacts, or functionally-zero nits — NOT because anything was fixed. Kept here for the audit trail._

- **[retracted-false-positive]** (android-ios) iOS screen infinite-loops / re-initializes — @StateObject wraps a shared singleton service and a .task with no id re-fires on every re-render ([ios] Maestro assertion failed [chat.yaml] on ios: 
Waiting for flows to complete...
[Failed] logistics-ios-chat-smoke (1s)

1/1 Flow Failed


)
  - _why:_ MISDIAGNOSIS — the stated mechanism cannot exist in this app. The finding blames SwiftUI `@StateObject` wrapping a singleton plus a `.task` without an id re-firing, but logistics mobile is a FLUTTER (Dart) app: no SwiftUI, no @StateObject, no .task. The observed iOS symptom traced to a stale simulator build — `flutter build ios` silently reinstalled the PREVIOUS Runner.app while ios/Flutter/ephemeral was read-only, so the app ran with the wrong API_BASE_URL and could not reach the backend. Not an app/skills defect.
- **[retracted-false-positive]** (android-ios) iOS: only the first call per app launch connects — after any completed call, the next accepted call never transitions to the ongoing screen (sticks at Calling / silently drops); kit-internal state breaks and only a full app restart recovers ([ios] Maestro assertion failed [calling.yaml] on ios: 
Waiting for flows to complete...
[Failed] logistics-ios-calling-smoke (1s)

1/1 Flow Failed


)
  - _why:_ CONTRADICTED BY OWN EVIDENCE — 'only the first call per app launch connects; only a full app restart recovers' does not reproduce. At CP2 the operator ran the full manual two-device matrix and 1:1 calls MULTIPLE (a second call placed after the first ends) passed on all three pairs, 18/18. A live android<->iOS second call was separately observed connecting in-session (accepted -> onParticipantJoined -> onSessionJoined) with no app restart.
- **[methodology-artifact]** (android-ios) Visual defect: Expected an incoming-call UI, but the screen shows the app's login screen instead — no call surface is present (screenshot /Users/admin/Desktop/orchestrator/cometchat-skills-reviewer/cometchat-skills-reviewer/_reports/2026-07-08-baseline/logistics/android-ios/smoke/android/shots/calling.yaml.png: Expected an incoming-call UI, but the screen shows the app's login screen instead — no call surface is present | judge quoted: '"Northwind Dispatch", "Sign in", "DEMO ACCOUNTS", "Tap a role to sign in — shared password \\"password\\"."')
  - _why:_ MISATTRIBUTION — produced by an UNAUTHORED smoke flow, not the app. As the harness-error blockers in this same report establish, smoke/flows/logistics/{android,ios}/calling.yaml contains only `launchApp: {clearState: true}`; clearState wipes the app to the login screen and the flow ends without signing in or navigating, so the judge necessarily saw a login screen. Measures the harness, not the skills under review.
- **[methodology-artifact]** (android-ios) Visual defect: Expected a chat thread with message bubbles, but the screen shows a route list (Route Board), not a conversation (screenshot /Users/admin/Desktop/orchestrator/cometchat-skills-reviewer/cometchat-skills-reviewer/_reports/2026-07-08-baseline/logistics/android-ios/smoke/android/shots/chat.yaml.png: Expected a chat thread with message bubbles, but the screen shows a route list (Route Board), not a conversation | judge quoted: '"Route Board", "rte-001", "2026-07-20 · driver usr-driver", "Planned", "2 stops"')
  - _why:_ MISATTRIBUTION — same unauthored-stub cause (chat.yaml is `launchApp` only). The identical stub yielded a login screen, 'Route Board', and a blank frame across three runs — dispositive that the FLOW, not the app, controlled the screen. The app wires a Chats tab on every role opening the real CometChatConversations UI Kit.

## Prioritized improvement suggestions

- **[skills]** (backend) Add a language-agnostic 'server-side REST provisioning' section to cometchat-production (or a docs-mcp page) with a non-JS example — minimally a raw HTTP contract (headers appId/apiKey, POST /users, POST /users/{uid}/auth_tokens -> data.authToken, POST /groups with owner + members.{admins,participants}, POST /groups/{guid}/members) plus a Java/JVM snippet — so JSP/servlet and other non-JS backends aren't left to hand-translate.
- **[skills]** (backend) Align the production skill's .env sample and CLI scaffold to name the server secret COMETCHAT_REST_API_KEY (fullAccess) for user/group/token operations, and clearly document the Auth-Key (authOnly) vs REST-Key (fullAccess) scope split so the var naming matches the scope table.
- **[docs-mcp]** (backend) Document how to enforce 1:1/hub isolation in CometChat (e.g. user blocking, restrictions, or webhook-based message gating) so 'never converse directly' RBAC intents can be truly enforced rather than only implied by group membership.
- **[docs-mcp]** (web) Update the docs-MCP react-uikit-quickstart bundle to v6: remove CometChatConversationsWithMessages and show the composed inbox+thread shape (CometChatConversations + CometChatMessageHeader/MessageList/MessageComposer) with the onItemClick → getConversationWith() selection pattern.
- **[docs-mcp]** (web) Add a dedicated react-calls doc bundle to docs-MCP covering the dual-SDK init order, <CometChatIncomingCall /> app-wide mounting, and the ccCallEnded/ccCallRejected teardown + clearActiveCall/endSession/leaveSession release path, so calling integration doesn't depend solely on the skill file.
- **[skills]** (web) In the cometchat-react-calls skill, document the region-typing union and the v4/v5 endSession-vs-leaveSession naming split, with the recommended validated-string cast and optional-chaining pattern, so integrators don't have to rediscover the workaround.
- **[skills]** (android-ios) In cometchat-flutter-v5-conversations/core, add an explicit recipe: before opening a 1:1 thread or placing a direct call, resolve the real User via CometChat.getUser(uid) (or pass the known display name) rather than constructing a stub `User(uid:, name:<UI label>)`. Include a note that the object's `name` is what the thread header and call title render.
- **[skills]** (android-ios) Add a cometchat-flutter-v5-calls section for the v5 raw Calls SDK on Flutter (since the prebuilt calls UI Kit is 4.x-bound): document the joinSession + SessionStatusListeners lifecycle and the mandatory callee-side teardown (dismiss call screen on onCallEndedMessageReceived, leaveSession, CometChat.clearActiveCall) needed so a second call can connect — the exact ground the iOS ghost-call / only-first-call-connects known issues occupy.
- **[skills]** (android-ios) In cometchat-flutter-v5-messages/conversations, document the failure/empty-state pattern for entering a group/user thread whose membership may not yet be provisioned (guard + user-visible 'conversation unavailable' fallback), mirroring the inbox EmptyState, so hand-constructed group/user targets degrade gracefully.

## docs-mcp — infra note (coverage UNMEASURED this run)

- (backend) No server-side Java/JVM example for the auth-token mint + user/role/group REST provisioning flow; the production skill covers only JS/TS frameworks, so the agent had to hand-translate the REST v3 contract to java.net.http. A legacy JSP/servlet (javax.*) backend is a common enterprise target and has no worked example.
- (backend) No guidance on enforcing 1:1 conversation isolation (hub/RBAC rules) at the CometChat level — provisioning users leaves default-open 1:1 messaging, and docs-mcp/skills give no pointer to blocking/restrictions to actually enforce a 'never converse directly' rule.
- (web) No calls/react-calls bundle exists in docs-MCP — only react-uikit-quickstart. The agent had to fall back to the cometchat-react-calls SKILL file for CometChatCalls.init order and call-teardown event wiring; docs-MCP contributed only the events page.
- (web) docs-MCP gave nothing useful on the endSession vs leaveSession naming difference across Calls SDK v4/v5; the agent had to access both defensively via optional chaining.
- (android-ios) Flutter v5 raw calls: the prebuilt cometchat_calls_uikit is 4.x-bound/incompatible with the v5 Calls SDK, forcing a hand-rolled call surface (joinSession + SessionStatusListeners + CometChatOngoingCallService + clearActiveCall teardown). No docs-mcp/skill content covers building the v5 raw Calls-SDK call UI and mandatory callee-side teardown for Flutter.
- (android-ios) Backend JSP/servlet auth-token minting (POST /cometchat/token → {appId,region,uid,role,authToken} using the server-only REST API key) is documented in the README/consumed by the client but no docs-mcp query was recorded and no server code is in the mobile diff, so the server side is asserted, not evidenced.