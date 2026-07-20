# CometChat Skills Review — delivery

**Batch:** `2026-07-08-baseline` · **skills@** `ac87450a6be3c0f3300ac953942b5c8dd1b70c33` · **platforms:** backend, web, android-ios
**Repo:** ArnavKataria-CometChat/cc-review-delivery · **PR:** https://github.com/ArnavKataria-CometChat/cc-review-delivery/pull/1

## Headline

- Build pass: **3/3** · Trigger: **1/3** · Variant: **2/3**
- Feature completeness (avg): **90.0%** · Ease (avg): **3.7/5**
- Hallucinations: **0** · docs-escapes: **0** · retries: **2**
- Findings by tag: {'agent': 3, 'skills': 4, 'harness-error': 2, 'SDK': 1, 'docs-mcp': 1}

## Per-platform

| Platform | Build | Variant | Completeness | Ease |
|---|---|---|---|---|
| backend | ✅ | ✅ | 90% | 5 |
| web | ✅ | ❌ | 90% | 4 |
| android-ios | ✅ | ✅ | 90% | 2 |

## Skills exercised

`cometchat-flutter-v6-conversations`, `cometchat-flutter-v6-messages`, `cometchat-react-calls`, `cometchat-troubleshooting`

> **Reading this report:** every finding below is a skill / docs / SDK gap observed in the baseline integration. Where a fix was applied to the demo app during review, that is a *verification aid only* — the underlying gap remains **open** until the skill, docs, or SDK is updated. Fix status never downgrades a finding.


## Findings — gaps by owner (11 total)


### skills — the review's primary product  (4)

- (android-ios) Android ghost call — the UI Kit's ongoing-call activity is not finished when the remote party ends a 1:1 call; onCallEndedMessageReceived is a default no-op the app never overrides, so the local user is stuck in a conference-of-one until they manually hang up (verified in source, not by the failing flow: the app never overrode CallListener.onCallEndedMessageReceived (a default no-op) until the fix; mobile/lib/cometchat/cometchat_service.dart now overrides it)
- (android-ios) Android call buttons untappable — the conversation screen renders edge-to-edge with the CometChat message header UNDER the status bar; the voice/video call buttons land in the status-bar touch region, so they are visible but untappable and a call can never be initiated (verified in source, not by the failing flow: the fix is in mobile/lib/cometchat/direct_chat_screen.dart (commit daa1ddc), a PreferredSize appBar reserving MediaQuery.padding.top, with a comment naming the status-bar touch region as the cause)
- (android-ios) Calling is in scope but no flutter-v6 CALLING skill was loaded (only conversations + messages). The agent had to hand-roll the entire calling path from SDK source reads: the separate CometChatUIKitCalls.init, enableCalls=true, the CallNavigationContext.navigatorKey wiring in main.dart, re-arming calls after logout, and all native setup (Podfile platform 15.1 + clearing EXCLUDED_ARCHS for the arm64 simulator slice, android minSdk 26, camera/mic/bluetooth permissions). This is a skills coverage gap distinct from the specific runtime overlay bugs.
- (android-ios) Flutter UI Kit: the CALLER cannot learn its call was answered. CometChatCallEvents.ccCallAccepted fires on the CALLEE only; the caller must subscribe to the raw SDK's CallListener.onOutgoingCallAccepted, which no skill or doc mentions. Without it a caller backgrounding a LIVE call believes the call is still ringing and sends rejectCall(cancelled), which the server refuses ('cannot update from ongoing to cancelled'), stranding the session. Compounds F1: together they make an unreleasable session the DEFAULT outcome of leaving an app mid-call.

### docs-mcp — documentation coverage  (1)

- (android-ios) Flutter UI Kit docstring is actively misleading: CometChat.clearActiveCall() is documented in call_event_service.dart:604 as 'Always clears the SERVER-SIDE state', but it only clears a device-local cache. It is the API an integrator reaches for when calls start returning busy, and it silently no-ops against exactly that state. This misdirection is plausibly why this whole class of bug survives review.

### SDK — CometChat SDK packaging / API  (1)

- (android-ios) Flutter SDK: CometChat.endCall() can NEVER end a connected call — it omits the server-required `joinedAt`, so the server rejects it and the session stays `ongoing` forever. CometChat then busy-rejects every later call for that user, to ANY peer, permanently; the state is server-side so it survives logout, reinstall and a device wipe. Measured on device: `endCall ERR The joinedAt post parameter is required to end a call`. Every layer beneath the public API accepts the parameter (updateCallStatusRaw(id,status,{joinedAt}), CallsApi.updateCallStatus(id,status,joinedAt)) and the Android SDK carries it — the Flutter port dropped it. Any app whose call ends by anything other than the kit's own hang-up button (backgrounded, evicted, force-quit, crashed) permanently bricks calling for that user. Worse, it is invisible from the console: the dashboard's Calls view reads the WebRTC log, which correctly shows `ended`, while the busy check reads the chat-side call record, which is stuck at `ongoing`. Workaround verified: PUT {appId}.apiclient-{region}.cometchat.io/v3.0/calls/{sessionId} {"status":"ended","joinedAt":<unix>} with appId + a USER authToken.

### agent — integrating-agent behaviour  (3)

- (backend) On FIRST group creation, syncOrderThread relies on POST /groups carrying an inline `members: { participants }` payload to seat the customer + assigned courier. If the v3 Create Group endpoint does not honor inline members (members are normally added via POST /groups/{guid}/members), the group is created empty and participants are never seated, because the reconciling addMembers() call only runs on the ERR_GUID_ALREADY_EXISTS branch. Unverified against docs; would leave freshly-created order threads with no members.
- (backend) Calling is in requested scope but the backend performs no calling-specific work — it only provisions users/groups/direct-peer UIDs (identical to chat provisioning). That is all a backend can do, but it means end-to-end voice/video is entirely dependent on client components that are out of this component's diff; there is no backend evidence calling actually functions.
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
- **[skills]** (android-ios) Add a dedicated cometchat-flutter-v6-calling skill (or a calling section in the messages skill) that documents: the separate CometChatUIKitCalls.init step, enableCalls=true, the required MaterialApp navigatorKey = CallNavigationContext.navigatorKey, re-init-after-logout, and the full native setup checklist (android minSdk 26 + calling permissions + network_security_config, iOS Podfile platform 15.1 + clearing EXCLUDED_ARCHS[sdk=iphonesimulator*] for arm64 sims, NSCamera/NSMicrophone usage strings). This is what the agent reverse-engineered from source.
- **[skills]** (android-ios) In the calling skill, add an explicit note that CometChatUIKitCalls.init failure must be surfaced (not swallowed) — recommend disabling/hiding the call buttons when the calls SDK is not initialized rather than completing init as success.

## docs-mcp — infra note (coverage UNMEASURED this run)

- (backend) No docs-mcp query backed the server-side provisioning REST surface actually used: POST /users, POST /users/{uid}/auth_tokens, POST /groups (with inline members), POST/DELETE /groups/{guid}/members, GET /groups/{guid}/messages. The agent relied on prior knowledge for these rather than docs, so their exact request/response shapes are unverified against docs-mcp.
- (web) An empty-string docs-mcp query was issued (no useful retrieval), indicating either an agent query-construction artifact or a docs bundle that returns nothing on degenerate input.