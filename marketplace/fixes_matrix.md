
### 2026-07-13 — recorded via record_finding.py

| Component | Tag | Finding |
|---|---|---|
| android | skills | Incoming-call overlay covers the conversation header (android). CometChatIncomingCall (UIKit v6.0.3) was embedded as an always-mounted match_parent overlay on the assumption it 'renders nothing until a call arrives'; it actually renders its Accept/Decline chrome unconditionally (no bound call), permanently covering the message header — peer identity, back button, and the voice/video call buttons — so a call can never be started from the thread. Verified with a freshly-provisioned user in a zero-history thread (0 incoming-call events in logcat) and with both parties of a 1:1 seeing it simultaneously. FIXED: keep the widget GONE and gate visibility on an incoming-call StateFlow<Call?> (set on onIncomingCallReceived, cleared on cancel/reject/accept/end) — the pattern the telehealth build used. |

### 2026-07-13 — recorded via record_finding.py

| Component | Tag | Finding |
|---|---|---|
| android | skills | Placing a call crashes with 'Please call the CometChat.init() method ... preferably in the onCreate() method of the application class'. The app initializes the CometChat SDK lazily (ChatManager.ensureReady, on first chat open) instead of in Application.onCreate — MarketplaceApp.onCreate only calls ChatManager.install(), never CometChat/UIKit init. CometChat's calling UI runs in a SEPARATE PROCESS where the SDK was never initialized, so a UIKit view crashes: CometChatMessageComposer.onDetachedFromWindow -> CometChatMessageComposerViewModel.endTyping -> CometChat.getLoggedInUser -> SQLiteManager.getInstance throws (RuntimeException), taking down the call process. Confirmed: crash PID differed from the main app PID. FIX: cache the non-secret App ID + Region on first backend token fetch and call CometChatUIKit.init + CometChatCalls.init in Application.onCreate (runs in every process) so the SDK is initialized wherever a CometChat view mounts. |

### 2026-07-13 — recorded via record_finding.py

| Component | Tag | Finding |
|---|---|---|
| android | skills | Android calling is impossible from the UI: the conversation screen renders edge-to-edge with the CometChat message header UNDER the status bar (no window insets applied). The header's voice/video call buttons sit in the top ~130px region where the system status-bar window consumes touches, so the buttons are untappable and a call can never be initiated. Confirmed live: tapping the voice button (bounds [891,45][954,108]) did nothing — no call initiated, an ONLINE 2nd client received no incoming call, no OutgoingCall event. After applying android:fitsSystemWindows=true to the ChatActivity root (header moves below the status bar, button now [891,181][954,244]), the SAME tap initiates the call: logcat 'Call Initiated Successfully', CometChatCallActivity (outgoing screen) mounts ringing the peer, and hang-up returns to the app. FIXED. |

### 2026-07-14 — recorded via record_finding.py

| Component | Tag | Finding |
|---|---|---|
| web | agent | Web CometChat chat/calling UI is MISSING from the committed feature-branch source. The running docker web build serves a full working chat (CometChat conversations, 1:1 message header with voice/video call buttons, message list/composer — .cometchat-* DOM), but git 'diff main..feature/cometchat-integration -- web' shows ONLY package.json/lock, api/endpoints.ts, api/types.ts, cometchat/errors.ts, cometchat/ids.ts changed — NO chat component, NO @cometchat/chat-uikit-react import, NO /messages route, NO CometChatMessages/MessageHeader anywhere in web/src (34 tsx/ts files, all grep-clean). So the deliverable web app was built from an uncommitted/lost version; rebuilding from HEAD yields the SDK deps + token endpoint but NO chat. Likely lost when the Phase B re-integration hit the usage limit and resumed with --components android,ios (per HANDOFF), skipping a web re-commit. Additionally, the running build's dispute-GROUP conversation header has NO call buttons (group voice/video calling absent on web) though 1:1 does. |

### 2026-07-14 — recorded via record_finding.py

| Component | Tag | Finding |
|---|---|---|
| ios | agent | iOS dispute-GROUP chat is unreachable: tapping 'Open group chat & call' in ReportDetailView pops back to the Disputes queue instead of pushing the group ChatScreen — so group chat + group voice/video calling can never be opened on iOS. Reproduced 4x live (support login -> Disputes -> open a FLAGGED report whose inquiry is escalated -> scroll to the 'Dispute group' section -> tap 'Open group chat & call' -> lands back on the Disputes queue, two levels up). The 1:1 'Conversation' NavigationLink (InquiryDetailView -> ChatScreen(target:.user)) pushes correctly, so maestro/XCUITest CAN follow this app's NavigationLinks — the group link (ChatScreen(target:.group(guid:'dispute-<id>'))) specifically fails. Likely a SwiftUI navigation issue (the conditionally-rendered 'if inquiry.flagged' section's NavigationLink auto-popping on a data refresh, or the group ChatScreen throwing on connect). Root cause not yet isolated; behavior (group unreachable) is confirmed. |

### 2026-07-14 — recorded via record_finding.py

| Component | Tag | Finding |
|---|---|---|
| web | agent | Gate 6 (cross-platform interop): a 1:1 VOICE call android-buyer -> web-seller does NOT connect. Signaling works — the caller's outgoing surface mounts, the web callee receives the incoming-call popup (.cometchat-incoming-call with __button-accept/__button-decline) and answering it (the REAL accept button, verified both via the Gate 6 harness AND a manual click) — but the media session never establishes: no ongoing-call UI mounts on either side, the android caller returns to chat, and the web call is logged as a 'Missed Call'. Reproduced 2x via gate6/run.py (choreography marketplace-1to1-voice fails at step 4 'call_connected', 30s timeout on BOTH clients) + once manually. The web client repeatedly logs the CometChat console error 'uiKitSettings not available' during call init — a concrete signal the web calling UIKit settings are not fully initialized (consistent with W5: the web CometChat chat/calling UI is missing from committed source; the running docker build is from an uncommitted/lost version). CAVEAT: a headless-Chromium <-> android-emulator WebRTC media path over docker/loopback could also contribute; the uiKitSettings signal + W5 point at incomplete web integration as the primary suspect. |

### 2026-07-14 — recorded via record_finding.py

| Component | Tag | Finding |
|---|---|---|
| web | skills | CometChat UIKit React v6 renders every component functional-but-UNSTYLED (no message bubbles, avatars/icons/call-buttons collapse to blank boxes, composer looks broken) when the app does not import the kit's design-token stylesheet '@cometchat/chat-uikit-react/css-variables.css'. The chat is fully operable (messages send/receive, calls place) so the failure is purely visual and silent. |

### 2026-07-14 — recorded via record_finding.py

| Component | Tag | Finding |
|---|---|---|
| android | skills | Group/conference calls stay stuck on 'connecting' (spinner never resolves) because CometChatCalls.joinSession runs before the calls CORE (CometChatCalls.init) is initialized in the current session, throwing 'Please call the CometChatCalls.init() method ...'. Root cause: (1) the eager per-process init bailed whenever the CHAT SDK was already initialized, never (re)initing the calls core; (2) the guard CometChatUIKit.isCallsSDKInitialized() reads true after UIKit setEnableCalling(true) even when CometChatCalls.init was never actually called this session, so the init was skipped. 1:1 calls work (they go through the chat SDK) which masks it — only the group/conference joinSession path exposes it. |

### 2026-07-14 — recorded via record_finding.py

| Component | Tag | Finding |
|---|---|---|
| android | skills | 1:1 (and group) call UI breaks after minimise+reopen: the call renders correctly at first, but pressing home enters Picture-in-Picture (the UIKit declares CometChatCallActivity/CometChatOngoingCallActivity with supportsPictureInPicture=true), which resizes the React-Native/Jitsi call surface to the small PiP window; because those activities also declare configChanges=screenSize\|screenLayout\|orientation they handle the resize themselves and the RN surface never re-measures on return to full screen. Result: the whole call UI (participant tiles + controls) stays crammed in a narrow left strip with the rest of the screen black, while the call is actually still connected. |

### 2026-07-14 — recorded via record_finding.py

| Component | Tag | Finding |
|---|---|---|
| android | agent | Placing or accepting a call intermittently 'redirects to the chats page': the app's own A3/A4 workaround (re-foreground MainActivity when a call ends) ran on ANY 'call ended' event/message with no session check, so a stale end message from an OLD unanswered call (e.g. the other side reloading their tab cancels its ring) arriving right as a NEW call starts brought the app task (top = Conversations) in front of the just-opened call screen. Compounding: the kit call activity's onUserLeaveHint then pushed the live call into PiP, and on return its RN/Jitsi surface stayed at PiP size (narrow-strip UI, A10). |

### 2026-07-14 — recorded via record_finding.py

| Component | Tag | Finding |
|---|---|---|
| ios | skills | iOS chat stuck on 'Connecting to chat…' after a cold login: ChatService.connect ran inside SwiftUI .task blocks, and the CometChat SDK init/login use withCheckedThrowingContinuation which does NOT observe Swift Task cancellation. When the hosting view updated (cancelling its .task) while suspended in a continuation, the continuation was orphaned — the await never returned, phase stuck at .connecting forever, POST /cometchat/token logged as cancelled. |

### 2026-07-14 — recorded via record_finding.py

| Component | Tag | Finding |
|---|---|---|
| ios | skills | iOS CometChatMessageList renders skeleton loaders forever when a 1:1/group conversation is opened after a cold login: login succeeds (phase .ready), getUser/getGroup resolve (header+composer render), and the conversation LIST loads — but the message-HISTORY fetch never completes, so the thread shows shimmer placeholders indefinitely. The same accounts/conversations load fine on web. Re-setting the subject, adding a ConnectionListener re-fetch, and calling messageList.reload() do NOT recover it. |

### 2026-07-14 — recorded via record_finding.py

| Component | Tag | Finding |
|---|---|---|
| ios | skills | iOS CometChatMessageList shows skeleton loaders forever (no timeout/error) when the conversation SUBJECT is set AFTER the list is already on-screen. The list fires its history fetch once, when it enters the window, ONLY if a subject (user/group) is already set. A common integration — create the list, add it to the view, then set the subject in an async getUser/getGroup callback — misses that window and never fetches, so the thread hangs on skeletons even though the socket is connected (calls deliver fine). Setting the subject synchronously from the known id before the view appears fixes it. |

### 2026-07-14 — recorded via record_finding.py

| Component | Tag | Finding |
|---|---|---|
| web | skills | Calls between clients fail immediately as 'Call Busy' / disconnect the instant they are initiated when EITHER side has a STALE/ghost active call in the SDK (a call surface that did not tear down, an app killed mid-call, or a lingering session). CometChat reports that user BUSY across ALL their sessions, so every new incoming call auto-rejects as 'Call Busy'. Neither the web nor the mobile UIKit clears a stale active call on (re)login, so a user can get permanently stuck 'busy' until the SDK state is reset. Aggravated by the same user being logged into multiple clients — any one busy/ghost session makes the user busy everywhere. |
