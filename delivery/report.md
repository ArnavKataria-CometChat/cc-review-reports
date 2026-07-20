# CometChat Skills Review — delivery

**Batch:** `2026-07-08-baseline` · **skills@** `ac87450a6be3c0f3300ac953942b5c8dd1b70c33` · **platforms:** backend, web, android-ios
**Repo:** ArnavKataria-CometChat/cc-review-delivery · **PR:** https://github.com/ArnavKataria-CometChat/cc-review-delivery/pull/1

## Headline

- Build pass: **2/3** · Trigger: **1/3** · Variant: **2/3**
- Feature completeness (avg): **90.0%** · Ease (avg): **3.7/5**
- Hallucinations: **0** · docs-escapes: **0** · retries: **2**
- Findings by tag: {'agent': 6, 'skills': 5, 'docs-mcp': 1, 'harness-error': 2}

## Per-platform

| Platform | Build | Variant | Completeness | Ease |
|---|---|---|---|---|
| backend | ✅ | ✅ | 90% | 5 |
| web | ✅ | ❌ | 90% | 4 |
| android-ios | ❌ | ✅ | 90% | 2 |

## Skills exercised

`cometchat-flutter-v6-conversations`, `cometchat-flutter-v6-messages`, `cometchat-react-calls`, `cometchat-troubleshooting`

> **Reading this report:** every finding below is a skill / docs / SDK gap observed in the baseline integration. Where a fix was applied to the demo app during review, that is a *verification aid only* — the underlying gap remains **open** until the skill, docs, or SDK is updated. Fix status never downgrades a finding.


## Findings — gaps by owner (14 total)


### skills — the review's primary product  (5)

- (web) Web chat won't scroll — a custom overflow/height override on the kit wrapper clips the CometChat scroller (Playwright assert failed [chat.mjs]: W1: chat root collapsed to 63px)
- (android-ios) Android ghost call — the UI Kit's ongoing-call activity is not finished when the remote party ends a 1:1 call; onCallEndedMessageReceived is a default no-op the app never overrides, so the local user is stuck in a conference-of-one until they manually hang up ([android] Maestro assertion failed [calling.yaml] on android: 
Waiting for flows to complete...
[Failed] delivery-android-calling-smoke (1m 9s) (Assertion is false: "(?s).*Calling.*" is visible)

1/1 Flow Failed


)
- (android-ios) Android incoming-call overlay covers the conversation header — CometChatIncomingCall (UIKit v6) mounted as an always-on overlay renders its Accept/Decline chrome with no bound call, hiding the header (peer name, back, voice/video call buttons) so a call can never be started ([android] Maestro assertion failed [calling.yaml] on android: 
Waiting for flows to complete...
[Failed] delivery-android-calling-smoke (1m 9s) (Assertion is false: "(?s).*Calling.*" is visible)

1/1 Flow Failed


)
- (android-ios) Android call buttons untappable — the conversation screen renders edge-to-edge with the CometChat message header UNDER the status bar; the voice/video call buttons land in the status-bar touch region, so they are visible but untappable and a call can never be initiated ([android] Maestro assertion failed [calling.yaml] on android: 
Waiting for flows to complete...
[Failed] delivery-android-calling-smoke (1m 9s) (Assertion is false: "(?s).*Calling.*" is visible)

1/1 Flow Failed


)
- (android-ios) Calling is in scope but no flutter-v6 CALLING skill was loaded (only conversations + messages). The agent had to hand-roll the entire calling path from SDK source reads: the separate CometChatUIKitCalls.init, enableCalls=true, the CallNavigationContext.navigatorKey wiring in main.dart, re-arming calls after logout, and all native setup (Podfile platform 15.1 + clearing EXCLUDED_ARCHS for the arm64 simulator slice, android minSdk 26, camera/mic/bluetooth permissions). This is a skills coverage gap distinct from the specific runtime overlay bugs.

### docs-mcp — documentation coverage  (1)

- (web) docs-mcp / troubleshooting had no coverage for the manual-composition container-height scroller contract that caused the collapse; ground truth for the fix had to be reverse-engineered from the installed `@cometchat/chat-uikit-react/dist` bundle's inline styles rather than any skill or doc.

### agent — integrating-agent behaviour  (6)

- (backend) On FIRST group creation, syncOrderThread relies on POST /groups carrying an inline `members: { participants }` payload to seat the customer + assigned courier. If the v3 Create Group endpoint does not honor inline members (members are normally added via POST /groups/{guid}/members), the group is created empty and participants are never seated, because the reconciling addMembers() call only runs on the ERR_GUID_ALREADY_EXISTS branch. Unverified against docs; would leave freshly-created order threads with no members.
- (backend) Calling is in requested scope but the backend performs no calling-specific work — it only provisions users/groups/direct-peer UIDs (identical to chat provisioning). That is all a backend can do, but it means end-to-end voice/video is entirely dependent on client components that are out of this component's diff; there is no backend evidence calling actually functions.
- (web) Latent same-class defect left unfixed: the sibling kit scroller panes `.cc-members` and `.cc-messages-list` depend on the identical `height:inherit`/`height:100%` chain but were deliberately left untouched. If those panes are used (conversation-list inbox, group members), they will collapse the same way the message list did. The agent scoped the fix to only the named 'chat root' without hardening the equivalent scrollers.
- (web) The collapse→fixed transition was not verified against the actual Playwright detector (chat.mjs) — it requires live CometChat credentials absent from the repo. The fix rests on CSS/flexbox reasoning plus a passing tsc+vite build, not observed runtime behavior, so the scroll fix is unconfirmed end-to-end.
- (android-ios) build_pass is false in the AUTO facts, contradicting the integration summary's claim that 'both native build gates green.' The self-reported build success is not corroborated by the harness; the overclaim masks whether the apk/simulator builds actually succeeded.
- (android-ios) _initCalls swallows its onError and completes successfully ('non-fatal for chat'), so if the calling SDK fails to initialize the app proceeds with calling silently broken and no user-facing signal — voice/video buttons render but calls cannot connect.

### harness-error  (2)

- (android-ios) ios smoke drove the pre-installed app — the built sim app is --no-codesign (I9: keychain -34018 breaks CometChat login), so it is NOT auto-installed and freshness is UNVERIFIED ([ios] /Users/admin/Desktop/Allapplications/delivery/mobile/build/ios/iphonesimulator/Runner.app)
- (android-ios) HARNESS stale-install: could not install the build under test (app-debug.apk) on emulator-5554: Performing Streamed Install
adb: failed to install /Users/admin/Desktop/Allapplications/delivery/mobile/build/app/outputs/flutter-apk/app-debug.apk: Failure [INSTALL_FAILED_INSUFFICIENT_STORAGE: Failed to override installation location]

 (Performing Streamed Install
adb: failed to install /Users/admin/Desktop/Allapplications/delivery/mobile/build/app/outputs/flutter-apk/app-debug.apk: Failure [INSTALL_FAILED_INSUFFICIENT_STORAGE: Failed to override installation location]

)

## Prioritized improvement suggestions

- **[docs-mcp]** (backend) Add a docs-mcp REST-API v3 page (or skill note) that explicitly documents the server-side provisioning sequence — Create Group vs. Add Members semantics, specifically whether POST /groups accepts an inline members object or whether a separate POST /groups/{guid}/members call is mandatory to seat participants — with a canonical create-then-add-members snippet.
- **[docs-mcp]** (backend) Add a docs-mcp reference for the backend auth-token flow (POST /users then POST /users/{uid}/auth_tokens, response envelope shape, apikey header, base URL {appId}.api-{region}.cometchat.io/v3) so Node/Express integrations can verify the mint-token path instead of relying on prior knowledge.
- **[skills]** (web) Add a `cometchat-troubleshooting` entry titled 'Chat root collapses / message list won't scroll (hand-composed layout)': state that when embedding `CometChatMessageList`/`MessageHeader`/`MessageComposer` in an app-owned layout, every wrapper in the chain must expose a definite `height` (e.g. `height:100%` on a flex child, or a bounded flex parent) because the kit root renders with inline `height:inherit` and inner body uses `height:100%; overflow-y:auto`; `flex:1 1 0` alone does not resolve the percentage chain.
- **[docs-mcp]** (web) Add a docs-mcp page in the react-uikit bundle documenting the container-height contract for manual composition (header + list + composer), including the required definite-height chain and the `height:inherit` behavior of the kit root, so a docs query on 'message list won't scroll / container height' returns an authoritative answer instead of only the all-in-one quickstart component.
- **[skills]** (web) In the same troubleshooting entry, note that the definite-height requirement applies to ALL kit scroller panes — message list, conversation/inbox list, and members list — so integrators harden every embedded scroller rather than only the one that visibly broke first.
- **[skills]** (android-ios) Add a dedicated cometchat-flutter-v6-calling skill (or a calling section in the messages skill) that documents: the separate CometChatUIKitCalls.init step, enableCalls=true, the required MaterialApp navigatorKey = CallNavigationContext.navigatorKey, re-init-after-logout, and the full native setup checklist (android minSdk 26 + calling permissions + network_security_config, iOS Podfile platform 15.1 + clearing EXCLUDED_ARCHS[sdk=iphonesimulator*] for arm64 sims, NSCamera/NSMicrophone usage strings). This is what the agent reverse-engineered from source.
- **[skills]** (android-ios) In the calling skill, add an explicit note that CometChatUIKitCalls.init failure must be surfaced (not swallowed) — recommend disabling/hiding the call buttons when the calls SDK is not initialized rather than completing init as success.

## docs-mcp — infra note (coverage UNMEASURED this run)

- (backend) No docs-mcp query backed the server-side provisioning REST surface actually used: POST /users, POST /users/{uid}/auth_tokens, POST /groups (with inline members), POST/DELETE /groups/{guid}/members, GET /groups/{guid}/messages. The agent relied on prior knowledge for these rather than docs, so their exact request/response shapes are unverified against docs-mcp.
- (web) No docs-mcp page or troubleshooting entry documents the container-height contract for a hand-composed header+list+composer layout: the kit's message-list root renders with inline `height:inherit` and its inner `.cometchat-message-list__body` uses `height:100%; overflow-y:auto`, so an app wrapper with no definite `height` collapses the scroller to the header height (~63px). The quickstart bundle only demonstrates the all-in-one `<CometChatConversationsWithMessages />` and never covers manual composition.
- (web) An empty-string docs-mcp query was issued (no useful retrieval), indicating either an agent query-construction artifact or a docs bundle that returns nothing on degenerate input.