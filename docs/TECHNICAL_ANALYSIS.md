# Technical Analysis: MH4G / MH4U Charge Blade GP and Red-Shield Phial Burst

![Guard Point and phial-burst mechanism flowchart](../assets/gp-phial-burst-flowchart.svg)

The diagram gives the overall relationship first. The sections below explain when GP can actually block an attack, why GP reduces recoil, how the game confirms a successful contact, and how a red-shield burst reaches damage processing. These are four separate steps, not one switch.

## 1. Four mechanisms, not one switch

The early investigation repeatedly ran into the same trap: guard sparks, an attack being blocked, GP reducing recoil, and a red-shield phial burst were treated as if they came from one flag. V4 separated them:

```text
action reaches the point where GP can become active
→ the game prepares guard acceptance and GP recoil adjustment
→ monster attack contacts the shield
→ guard system decides whether the hit is actually blocked
→ recoil class and displacement are calculated
→ a successful red-shield GP enters the phial-burst object, collision, and damage path
```

This explains why copying a single observed state field produced only partial behavior. A field could affect guard acceptance or recoil without reconstructing the burst path, while changing a result Action could alter the aftermath without making a previously unavailable GP guard check valid.

## 2. When the GP guard check becomes active

On Japanese v1.2, `CA783C` checks the current action, action phase, and timing conditions. It then uses two shared helper functions, `B56160` and `B039B0`, to set or clear two important bits in `state+F0`:

```text
bit 0x10
bit 0x2000
```

During v3 research, logical motion ID `0x592` was the action identity that entered this timing path. Later v4 work showed that the final patch does not need to replace native fast morph's `0x583` identity with `0x592`. Fast morph already performs a usable `mask=2` state update at the right time. When the current player, target Action, requested mask, and pre-call state all match, v4 expands only that update to `0x2012` and then returns to the original `B56160` helper.

`0x2012` is not a newly invented all-in-one GP flag. It combines the three parts needed for that one state update:

```text
0x0002  mask already requested by native fast morph
0x0010  actual guard-acceptance condition
0x2000  GP recoil-classification adjustment
---------------------------------------------
0x2012  mask passed to the original helper
```

This is why the final implementation preserves the native `mask=2` timing instead of forcing two state bits to remain active. V4 adds the verified components only when the game is already making the relevant state update.

The precise conclusion is therefore:

- `0x592` helped identify the native GP identity and timing path;
- final v4 does not impersonate `0x592`;
- v4 reuses a native state-establishment moment already present in fast morph.

The original `0x592` path's ending is also understood statically. In the later phase, `CA783C` clears `0x10` and `0x2000` through `B039B0`. Its `close=-1` value is not a hidden frame count; it uses `AF1D90 → AEEBC4` to test whether the current action has ended naturally.

Final fast v4 no longer uses the `0x592` identity, so that `0x592` closing route must not be presented as proof that one specific Action or flag closes fast v4. Repeated runtime tests confirmed that the fast-action state is recovered after the action ends. The careful conclusion is that the patch activates GP during the correct native update and lets the action's existing lifecycle perform cleanup; this document does not invent a fast-specific closing switch that has not been isolated independently.

## 3. `bit 0x10`: whether the guard is actually accepted

`B54EE0` reads `state+F0` and checks `bit 0x10` at `B55074`. The caller at `B018A8` immediately uses the return value to decide whether processing can continue as a valid guard.

A reversible call wrapper temporarily hid `bit 0x10` only while calling the original `B54EE0`, leaving all other state changes intact. The result was causal and reproducible:

- fast morph still showed guard-contact sparks, but the hunter was hit;
- held-R guarding also showed sparks, but no longer blocked the attack;
- restoring the original call restored both forms of guarding.

`bit 0x10` is therefore a key guard condition shared by held-R shield guarding and GP. It is not a GP-only marker. Guard sparks belong to an earlier or separate presentation layer: they show that contact occurred, but do not prove that the attack was blocked.

## 4. `bit 0x2000`: GP guard-performance adjustment

`B2804C` calculates an internal value that the game later uses to choose small, medium, or large recoil. It reads `state+F0` and checks `bit 0x2000`:

- without the bit, the value is retained;
- with the bit, the value is reduced by 10 and clamped to a minimum of 1.

This “-10” is not hunter defense, damage, stamina, or animation frames.

Two confirmed code paths use the result:

- `B0E5B0` selects small (0), medium (1), or large (2) recoil using action-dependent thresholds;
- `B23B48` selects recoil displacement values 150, 200, or 360.

This matches the earlier dynamic tests: `0x2000` alone cannot create a guard, but together with `0x10` it improves the final recoil result—the behavior players describe as GP's guard-performance bonus.

`B2804C` also uses generic queries 98, 99, and 100, but their player-visible names remain unknown. This document does not guess those names; the gap does not affect the verified `0x2000` path.

## 5. Red-shield phial burst is a separate successful-contact path

Bits `0x10` and `0x2000` do not automatically produce a phial burst.

After a successful fast-morph GP, v4 continues only when the Action, contact object, and guard result all match. It then checks the player's red-shield timer at `state+0x60`:

- timer is zero: keep the GP, but do not request a burst;
- timer is nonzero: set `state+0x489 bit 2`, handing this contact to the game's native burst path.

The confirmed Japanese-build downstream path is:

```text
CA895C sets state+0x489 bit 2
→ B3791C / CA92A8
→ type-3 phial-burst object
→ embedded active collision record
→ BD364C collision registration
→ BDA990 / BD42B4 entity resolution
→ BCFBEC / 919D0C hit-result submission
→ 909930 result-queue processing
→ 925128 target-current-HP update
```

The burst is therefore not merely a visual effect. Its object, collision, entity, result queue, and final damage are parts of one traceable native pipeline.

## 6. Fast Morph GP v4: two native-timing hooks

The standalone patch needs two hooks:

1. **GP timing hook:** for the current player's fast-morph Actions, expand a native mask-2 call with pre-call `F0=1` to `0x2012`, then execute the original helper.
2. **Successful-contact hook:** only on the successful fast-GP result path, check the red-shield timer and request the native burst when appropriate.

Target Actions are:

```text
000B0400  fast morph without moving the left stick
001C0400  fast morph while moving the left stick
```

The implementation does not write the motion ID, impersonate the native morph action, perform per-frame state writes, or include loggers. Japanese v1.2 uses two hooks and stores the patch logic in a 264-byte blank code area, commonly called a code cave. The IPS contains three records: two hooks and the cave. USA and EUR use the same behavior, with addresses and branches relocated independently for each build.

## 7. Why sword-mode charge GP needs different result handling

The sword-mode charge-hold Action is `00340400`. Very short or automatic release may enter `00330400`, while a normal release may enter `00220400`. A bounded logger showed that the usable native mask timing occurs during `00340400`, while the shield is visibly held forward, rather than during the later release attack.

The shared GP hook can establish `0x10` and `0x2000`, but the result must be handled by recoil class:

- small/medium recoil: suppress the result transition that would interrupt charging;
- large recoil: allow the native result to interrupt the charge.

Charge slash does not read `state+0x489 bit 2` at the same point as fast morph. Leaving that bit set for later could trigger it at the wrong time. The final implementation therefore calls the version-specific native phial-burst submitter directly after confirming the charge Action, successful guard result, and active red shield.

This still creates the game's own complete type-3 burst object and native collision/damage path; it is not a custom visual effect.

## 8. Combined-package structure

Both features need the same GP timing entry, so their independent IPS files cannot simply be concatenated. The combined package uses one shared dispatcher:

```text
shared GP hook
├─ 000B0400: fast morph without moving the left stick
├─ 001C0400: fast morph while moving the left stick
└─ 00340400: sword-mode charge hold

fast successful-contact hook
└─ enter the native bit-2 burst consumer when red shield is active

charge-result hook
├─ small/medium recoil: continue charging
├─ large recoil: native interruption
└─ red shield: call the native burst submitter directly
```

The combined release uses three hooks and a 792-byte cave. It already contains Fast Morph GP v4 and must not be installed together with the standalone patch.

## 9. Evidence status

### Verified

- `bit 0x10` is a shared guard-acceptance condition for held-R guarding and GP.
- `bit 0x2000` reduces the internal recoil-classification value by 10, clamped to 1.
- recoil class and physical displacement consume that value in separate functions.
- the red-shield burst has a separate object, collision, result-queue, and HP-damage pipeline.
- the two-hook native v4 preserves action identity and passed gameplay testing in all three regions.
- small/medium charge-GP recoil continues charging, while large recoil interrupts it, in all three regions.
- both standalone and combined releases passed installation checks, state checks, and safe restoration.

### Not exhaustively covered

- multi-hit monster attacks did not receive a dedicated test matrix;
- the player-visible names of queries 98/99/100 remain unknown;
- no repeated burst was observed, but this is not an exhaustive proof for every multi-hit timing pattern.
- reusing a native state-update point is not evidence of an official disabled “hidden fast GP” feature; the evidence supports this project's reconstruction and extension of native mechanisms.

## 10. The historical value of v3

V3 was not a “wrong version.” It was the first complete engineering solution for fast GP, red-shield burst, both input branches, repeated use, and state recovery, and it provided a stable A baseline for v4 research.

Its `0x592 → 0x583` overlay changed logical identity and resource lifetime together, so it proved that a particular combination was sufficient. V4 used later A/B tests, code paths that read the state bits, contact-path tracing, and transfer to a new action to separate that combination into independently understood native mechanisms and build a smaller final implementation.
