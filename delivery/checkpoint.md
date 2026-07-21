# delivery — checkpoint brief

# delivery — CometChat integration matrix

Rows are platform PAIRS, columns are capabilities. Every cell is a
two-client interop check: a single-device test cannot see a delivery
failure, because it only proves the sender saw their own action.

`calls single` proves a call connects once. `calls multiple` places a
SECOND call after the first ends, which is the only way to prove the app
RELEASES the session — an unreleased one pins the user server-side and
every later call returns busy.

✅ pass  ❌ fail  🟡 not verified

## Harness-verified (Gate 6)

```
             1:1             1:1             1:1             group           group           group           
             chat            calls single    calls multiple  chat            calls single    calls multiple  
-------------------------------------------------------------------------------------------------------------
web-android         ✅              ✅              ✅              ✅              ✅              ❌       
android-ios         ✅              ❌              ❌              ✅              ❌              ❌       
ios-web             ❌              ❌              ❌              ✅              ❌              🟡       

✅ pass   ❌ fail   🟡 not verified
8/18 passing, 17/18 run (1 never attempted)
```

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

- ⚠️ user cust-005: latest substantive call is `ongoing` (v1.in.1680958092adbe18e.17846229206fff2d0f7b2a9382be5433e15c784d0fc1f8f92b) — does not pin by itself, but if a LIVE client still holds it calls are genuinely busy; ensure no stray client is running.
- ⚠️ user cour-003: latest substantive call is `ongoing` (v1.in.1680958092adbe18e.17846229206fff2d0f7b2a9382be5433e15c784d0fc1f8f92b) — does not pin by itself, but if a LIVE client still holds it calls are genuinely busy; ensure no stray client is running.

## Open questions for the operator

_None — every cell verified green._

