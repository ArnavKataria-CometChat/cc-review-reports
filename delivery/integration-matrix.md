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
web-android         ✅              ✅              ✅              ✅              ❌              ❌       
android-ios         🟡              🟡              🟡              🟡              🟡              🟡       
ios-web             🟡              🟡              🟡              🟡              🟡              🟡       

✅ pass   ❌ fail   🟡 not verified
4/18 passing, 6/18 run (12 never attempted)
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
