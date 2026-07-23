# logistics — checkpoint brief

# logistics — CometChat integration matrix

Rows are platform PAIRS, columns are capabilities. Every cell is a
two-client interop check: a single-device test cannot see a delivery
failure, because it only proves the sender saw their own action.

`calls single` proves a call connects once. `calls multiple` places a
SECOND call after the first ends, which is the only way to prove the app
RELEASES the session — an unreleased one pins the user server-side and
every later call returns busy.

✅ pass  ❌ fail  🟡 not verified

## Operator-verified (manual, at CP2)

```
             1:1             1:1             1:1             group           group           group           
             chat            calls single    calls multiple  chat            calls single    calls multiple  
-------------------------------------------------------------------------------------------------------------
web-android         ✅              ✅              ✅              ✅              ✅              ✅       
android-ios         ✅              ✅              ✅              ✅              ✅              ✅       
ios-web             ✅              ✅              ✅              ✅              ✅              ✅       

✅ pass   ❌ fail   🟡 not verified
18/18 passing, 18/18 run (0 never attempted)
```


## Environment (checked live at generation time)

- ✅ all matrix platforms drivable

## Matrix account health

- ⚠️ user usr-driver: latest substantive call is `ongoing` (v1.in.1681084e28518c82b.1784797844ce9dddf9f4b19659cae9753e0028858e9c3bb122) — does not pin by itself, but if a LIVE client still holds it calls are genuinely busy; ensure no stray client is running.
- ⚠️ user usr-dispatcher: latest substantive call is `ongoing` (v1.in.1681084e28518c82b.1784797844ce9dddf9f4b19659cae9753e0028858e9c3bb122) — does not pin by itself, but if a LIVE client still holds it calls are genuinely busy; ensure no stray client is running.

## Open questions for the operator

- 🟡 **web-android / chat_1to1** — does a message sent on one platform arrive on the other WHILE the thread is open (both directions)? (yes / no / inprocess)
- 🟡 **web-android / call_1to1_single** — does one call connect end-to-end (ring → answer → media both sides)? (yes / no / inprocess)
- 🟡 **web-android / call_1to1_multiple** — after the first call ends, does a SECOND call still connect (session released)? (yes / no / inprocess)
- 🟡 **web-android / chat_group** — does a message sent on one platform arrive on the other WHILE the thread is open (both directions)? (yes / no / inprocess)
- 🟡 **web-android / call_group_single** — does one call connect end-to-end (ring → answer → media both sides)? (yes / no / inprocess)
- 🟡 **web-android / call_group_multiple** — after the first call ends, does a SECOND call still connect (session released)? (yes / no / inprocess)
- 🟡 **android-ios / chat_1to1** — does a message sent on one platform arrive on the other WHILE the thread is open (both directions)? (yes / no / inprocess)
- 🟡 **android-ios / call_1to1_single** — does one call connect end-to-end (ring → answer → media both sides)? (yes / no / inprocess)
- 🟡 **android-ios / call_1to1_multiple** — after the first call ends, does a SECOND call still connect (session released)? (yes / no / inprocess)
- 🟡 **android-ios / chat_group** — does a message sent on one platform arrive on the other WHILE the thread is open (both directions)? (yes / no / inprocess)
- 🟡 **android-ios / call_group_single** — does one call connect end-to-end (ring → answer → media both sides)? (yes / no / inprocess)
- 🟡 **android-ios / call_group_multiple** — after the first call ends, does a SECOND call still connect (session released)? (yes / no / inprocess)
- 🟡 **ios-web / chat_1to1** — does a message sent on one platform arrive on the other WHILE the thread is open (both directions)? (yes / no / inprocess)
- 🟡 **ios-web / call_1to1_single** — does one call connect end-to-end (ring → answer → media both sides)? (yes / no / inprocess)
- 🟡 **ios-web / call_1to1_multiple** — after the first call ends, does a SECOND call still connect (session released)? (yes / no / inprocess)
- 🟡 **ios-web / chat_group** — does a message sent on one platform arrive on the other WHILE the thread is open (both directions)? (yes / no / inprocess)
- 🟡 **ios-web / call_group_single** — does one call connect end-to-end (ring → answer → media both sides)? (yes / no / inprocess)
- 🟡 **ios-web / call_group_multiple** — after the first call ends, does a SECOND call still connect (session released)? (yes / no / inprocess)
