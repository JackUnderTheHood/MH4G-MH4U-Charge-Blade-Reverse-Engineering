# Research History: From v3 Identity Overlay to Native-Timing v4

This document does not reproduce the million-byte `current_state.md` line by line. It preserves the turning points that changed the investigation. Detailed candidates, failed builds, and operating notes remain in the archived research record. The raw `reports/` files referenced there belong to the local evidence set and are not all included in this public documentation package.

## 1. V3 completed the first full feature loop

V3 already delivered:

- fast morph both without moving the left stick and while moving it;
- fast animation, GP, red-shield burst, and primary follow-ups;
- morph/fast-morph action isolation;
- repeated use and state recovery;
- ExeFS releases for MH4G JPN/localized, MH4U USA, and MH4U EUR.

Its key observation was:

```text
morph: 0x592 → 0x583
native fast morph: 0x583
```

Five hooks and a 640-byte overlay gave the fast action the logical context associated with `0x592` for a bounded lifetime while retaining the visible fast resource from `0x583`. That combination was sufficient for the existing Guard and red-shield systems to produce the target behavior.

One mechanism question remained: why did the full `0x592` context produce GP plus burst, while the early native-fast state-bit experiment could guard but not burst?

## 2. 2026-08-13: strict A/B comparison

V4 research did not immediately alter the released v3. Instead, two conditions were alternated inside one controlled runtime:

- A: formal v3, with complete fast GP and red-shield burst;
- B: native fast identity plus the early guard-state experiment, which guarded but did not burst.

Samples were collected in short alternating batches and separated by recoil class. Any batch contaminated by a still-enabled experimental code was discarded.

The stable difference showed that both conditions could guard, while only A burst. Work therefore narrowed from broad post-hit field guessing to bounded differences between preparation timing, contact identity, and successful-result submission.

## 3. Removing noise from high-frequency traces

Small fixed-capacity loggers recorded only selected events: phase-0 calls, motion-getter callers, successful-contact identity, event dispatch, native call edges, and `state+0x489 bit 2` production or consumption.

Many important results were negative:

- some getters could compare `0x583/0x592`, but their actual call sites did not request `0x592`;
- some functions could theoretically create related side effects, yet two real red-shield burst samples never traversed the relevant native call edges;
- modifying a guard result or calling a candidate function after the result could not reconstruct the burst.

These controls prevented “the function can do this” from being rewritten as “this action depends on it.”

## 4. Identity A/B found two key reader groups

The experiment temporarily changed what the motion getter returned to selected callers without changing the real motion value in memory. Narrowing the caller list showed that one group of code needed to read `0x592` for GP, while another needed it for the red-shield burst. Keeping only one side produced either GP without burst or no accepted guard; keeping both restored full behavior.

However, the apparently symmetric candidate—starting from native `0x583` and returning `0x592` only to those two callers—failed. Animations remained correct, but GP did not appear.

Two bounded probe rounds then closed this route:

- **RC1** tried to raise `0x583` to `0x592` only for the known readers at `CA7864` and `CA8914`. One native fast action produced 2,777 getter calls that passed the common filters, but neither target caller appeared at all. Those callers occur only after the flow has already entered the `0x592` lifecycle; they cannot start that lifecycle from native `0x583`.
- **RC2** added the earlier `AEE784` identity-comparison site. It was reached 183 times, yet `CA7864` and `CA8914` remained at zero. Animation stayed normal, but GP was still absent.

The failure was therefore not a simple branch-encoding mistake. The assumption that getter-return aliases alone could bootstrap a complete `0x592` lifecycle was false. The project stopped enumerating getter callers and returned to the native state-establishment timing.

This failure was valuable: a static symmetry was not promoted into a runtime claim.

## 5. Finding an earlier native timing point

Logging the Action, requested mask, and pre-call state before `B56160` applied a mask showed that native fast morph already made a usable mask-2 call.

Strictly filtering the current player, fast Action, `mask=2`, and pre-call `F0=1`, then expanding only that mask to `0x2012`, gave native fast morph a GP without changing motion identity. This became the final v4 method for making the GP guard check active.

## 6. Separating guard acceptance from GP performance

On 2026-08-15, two causal links were completed:

1. Hiding `bit 0x10` only during `B018A8 -> B54EE0` left guard sparks visible but made both fast GP and held-R guarding fail to block the attack. Restoration recovered both.
2. Static and runtime checks showed `B2804C` consumes `bit 0x2000`, reducing the internal recoil-classification value by 10 (minimum 1), after which `B0E5B0` and `B23B48` select recoil class and displacement.

GP could now be described as native timing that establishes guard acceptance plus a separate GP recoil adjustment—not a single magic flag.

## 7. Closing the red-shield burst chain

Tracing from `state+0x489 bit 2` connected burst-object creation, embedded active collision, collision-list registration, entity resolution, hit-result accumulation, result-queue consumption, and final target-HP reduction.

The visible burst was therefore backed by a complete effect, collision, and damage chain.

## 8. The first “v4” repack was withdrawn

After the mechanism was documented, v3's five-hook bytes were briefly repackaged with v4 mechanism documents under a v4 filename. The user correctly rejected this as not being a true v4 implementation.

That archive was moved to history and marked:

```text
WITHDRAWN_NOT_TRUE_V4
```

It was never treated as the final v4 release. This explains an apparent contradiction in the raw history: “v4 equals v3 bytes” refers only to the withdrawn documentation repack; “v4 is a new two-hook implementation” refers to the final release.

## 9. Final Fast Morph GP v4

Final v4 uses two hooks: one at native mask timing and one at successful fast-GP contact for the red-shield condition. It preserves native Actions `000B0400` and `001C0400`, does not rewrite motion identity, and does not need v3's 640-byte overlay.

After Japanese acceptance, USA and EUR were independently mapped and tested for animations, non-red/red GP behavior, single bursts, follow-ups, held-R guarding, readback, and safe restoration.

## 10. Cross-validating the model with sword-mode charge slash

The next goal was to add GP to an action that did not originally have it: sword-mode charge slash.

Logging identified native mask timing during Action `00340400`, while the shield is held forward. At first, every successful guard switched to a recoil reaction and interrupted charging. Result observation then identified small, medium, and large recoil as substates 3, 4, and 5. The final patch blocks the small/medium transitions, 3 and 4, so charging continues, while leaving substate 5 to perform the game's normal large-recoil interruption.

Leaving `state+0x489 bit 2` pending risked delayed consumption, so the final charge implementation directly calls the regional native burst submitter after confirming the charge Action, successful result, and red-shield timer.

The new feature passed in all three regions. This demonstrated that the GP activation timing, recoil, and burst model could be reused outside the original fast-morph action and became the strongest cross-validation of the research.

## 11. Regional ports and the EUR crash confound

USA and EUR were mapped from their own runtime images. Functions, player-root slots, Actions, branches, and caves were verified independently; no uniform offset or renamed Japanese IPS was used.

During EUR testing, climbing twice crashed at `B96990`. Testing stopped because the patch could not initially be excluded. The user later disclosed a simultaneously enabled cheat that wrote a USA address, `00B96958`, into the EUR build, overwriting a required instruction in the EUR climbing function.

After a full cold restart removed that write, the same GP candidate remained installed and the same climbing route succeeded five consecutive times. The full feature matrix then passed. The crash was therefore attributed to the wrong-region cheat, not this patch. This episode remains documented because it demonstrates how an apparent patch crash was separated from an external confound.

## 12. 2026-08-16: formal completion

Six final packages were completed: standalone Fast Morph GP v4 and combined Charge Slash GP v1 for each of three regions. Every archive was deterministically built from runtime-tested machine code and independently rehashed.

The main research objective is closed. Dedicated multi-hit-monster coverage and the player-visible names of generic queries 98/99/100 remain explicit limits, but neither changes the verified GP/recoil/burst chain.
