# MH4G / MH4U Charge Blade Research History: From Single-Field Guesses to ExeFS v3 Covering Both Input Branches

This document organizes the work by causal phase. The complete turn-by-turn record is preserved in `../archive/current_state_research_archive.md`; the untouched pre-closure copy is `../archive/current_state_original.md`.

For the explicit division between AI analysis/implementation and human runtime testing/data collection, see [Development Methodology and Contribution Disclosure](DEVELOPMENT_METHODOLOGY.md).

The mechanism-discovery phase took place in the **localized MH4G v1.2 environment**. The later timeline also includes independent MH4U USA/EUR v1.1 relocation and acceptance work. Research logic, tooling, and experimental discipline were reused across builds; MH4G addresses and patch files were not.

Early “current conclusion” and “next step” sections in the archive describe their historical moment. They do not override the current release status in this history, the public README, and `RELEASE_NOTES.md`. The archive's prepended closure section remains authoritative for the MH4G main-research decision at its freeze point, while its then-current USA/EUR status was later superseded by the completed regional v3 releases.

## Enabling condition: Azahar 2126.0

The project could not be completed from the static listing alone. Action identity, motion sequences, resource-override lifetime, consecutive re-entry, and safe cleanup all required runtime evidence through GDB. Earlier Azahar versions could not maintain a sufficiently stable GDB connection for sustained capture, patch readback, controlled comparisons, and reliable restoration after failed experiments.

Azahar 2126.0 provided the first debugging baseline the project could depend on for repeated `preflight → install → read back → execute → inspect state → restore` cycles. From a research-engineering perspective, this was a critical infrastructure change that made the project viable and ultimately finishable. It is documented as the research/reproduction baseline; the final ExeFS mod does not require players to connect GDB, so it does not by itself establish the minimum Azahar version for normal play.

The archive records the explicit switch to 2126.0 on August 10. This enabling condition is placed before the chronological sections deliberately: it is the project's retrospective reproducibility baseline. Earlier unstable sessions produced useful partial observations, but they did not provide a dependable conventional debugging foundation on which safe, repeatable research could be published.

The improvement in 2126.0 was that the connection, pause control, and guest-memory access finally became stable enough to use; it did not make conventional breakpoints and watchpoints dependable as the primary reverse-engineering method. Breakpoints could still end in remote `E01` errors or failed continuation. Watchpoints repeatedly trapped on same-address, same-value writes in high-frequency state updates. Manual stepping also could not reproduce long action chains reliably. The project therefore moved to purpose-built GDB scripts and runtime instrumentation: preflight/enable/status/disable scripts managed experiment lifecycles, temporary code hooks and bounded loggers captured motion/Action sequences, snapshots and chunked exports supported offline analysis, and in-game GP, burst, animation, and follow-up behavior completed validation.

## 2026-08-03: Building a usable map of the action system

The work began with an approximately 197 MB, 3.4-million-line Ghidra listing. The project built a SQLite index, function extraction, cross-reference search, function comparison, and ARM32 branch encoding tools. This reduced the initial search to action creation, Action routing, and runtime state.

The first phase confirmed:

- `Player+0x11A8` as the current Action;
- fast-morph/morph Actions with no stick input, `000B0400/00060400`;
- `Player+0x28C` as the action frame and `Player+0x234` as freeze/resume control;
- `Player+0xA300` as the red-shield amount;
- red-shield phial burst as stronger GP evidence than the final Action alone.

The first partial positive was a state-field experiment rather than the final mechanism. At `state+0xF0`, `0x10` opened defense, `0x2000` alone did not, and `0x2010` together produced the improved knockback behavior expected of GP. The fast move could then defend while retaining its Action and animation, but red-shield contact still produced no phial burst. This proved that copying the observed defense bits recreated only part of GP, not the complete native path.

Directly replacing the fast Action, copying other candidate fields, or branching to a statically plausible handler likewise did not satisfy all requirements. Unstable remote connections, unreliable conventional breakpoints/watchpoints, and JIT code caching established the final method: use Azahar 2126.0 as the reproducible transport and memory-access baseline, but collect evidence primarily through automated scripts, temporary hooks, bounded loggers, snapshots, machine-code readback, and in-game causal controls.

## 2026-08-04 to 08-05: Ruling out a hidden burst switch in hit results

The project compared morph GP, fast-morph hits, red-shield on/off state, Guard-result scripts, retained event slots, collision objects, and transient fields. The important outcome was not a new switch but several exclusions:

- a final group-5 Action can be overwritten by a later hit and cannot prove GP by itself;
- red-shield state explains whether a burst appears but cannot create a missing Guard window;
- many morph/fast-morph differences in the result layer are consequences or correlated state, not the GP cause;
- no observed persistent flag could simply be copied into the fast move mid-action.

Two particularly useful negative controls were changing the fast Guard result from `4` to the morph result `3`, and injecting the Charge Blade event `0x51` after a Guard result. The first reproduced the result Action without a burst; the second also produced no burst. Clean red-shield on/off snapshots likewise found no stable derived Boolean that could be copied. These tests separated defensive result handling and visible burst consequences from the earlier action/motion cause.

The main line therefore moved from “what happens after contact” back to “how initialization and motion lifetime place the Guard system in the correct context.”

## 2026-08-06: First separation of logic and animation

Running the fast active handler with `mode=0` first produced the morph animation, GP, and red-shield burst. This was the key causal turn: morph logic could produce the desired defense, but direct reuse destroyed the fast visual identity.

Tracing the shared handler and submission path showed that `0x592` entered the generic chain as a complete motion/resource identity rather than a local Boolean.

The target changed from “copy the morph handler” to “preserve morph logic identity while substituting fast visual resources.”

## 2026-08-07: `0x592`/`0x583` and the first complete positive prototype

Static and dynamic analysis of motion resource tables, root objects, and submission functions confirmed independent resources for `0x592` and `0x583`. Combining the `0x592` logical context with fast resources produced the first simultaneous result of:

```text
fast Action
+ fast animation
+ native GP
+ red-shield phial burst
+ primary follow-ups
```

The phase and descriptor experiments around that prototype established that:

- phase 0 had to remain active through the defensive contact for GP;
- phase 1 also carried visible transition, sound, and follow-up logic and could not be deleted wholesale;
- same-tick or early phase-1 submission made the logical context too short for GP;
- duplicate submission, sentinel replacement, or crude phase removal caused double animation, missing transition, broken follow-ups, or crashes.

This proved the overall direction, but a global resource alias also changed the morph into the fast animation. Restoring too early recreated double animations. The problem became one of scope and lifetime: only the fast entry should establish the override, and it had to survive until the correct action boundary.

## 2026-08-08: Replacing function-name guesses with motion logging

A low-noise logger on the shared submission path recorded:

```text
morph:      592 → 583
native fast:     583
```

This reduced the critical difference from a large set of shared functions to motion identity and resource lifetime. The project then paired fast-morph and morph entry tails and designed an overlay established only by the fast entry and restored through motion/action boundaries.

The broader lesson was that a move visible to the player does not necessarily correspond to a dedicated function. Action, entry parameters, motion, resources, collision, and results need separate instrumentation.

## 2026-08-09: Converging from unstable prototypes to v6

Several overlay revisions exposed three engineering problems:

- re-entrant versions could crash, requiring exact LR, literal, and return-boundary verification;
- a dormant marker could make a later morph `592` use fast resources;
- clearing on every non-`583` motion removed the fast move's own required initial `592`.

Bounded motion logging produced the decisive prefixes:

```text
standalone fast:    592 → 583
fast GP follow-up:  592 → 592 → 583 → 579 → 56B
later morph:        567 → 592
```

v6 introduced the A/B/C prefix state machine: valid fast prefixes continue, while the morph `567` clears a dormant marker before the morph `592`. Strict consecutive GP, morph-animation isolation, red-shield bursts, knockback consistency, axe slam, roundslash, discharge, and state recovery all passed.

The same day, the 640-byte v6 overlay was packaged as Azahar ExeFS v2 and completed no-stick-input four-hook automatic-load and extended testing.

## 2026-08-09: The stick-input branch explains the old intermittent failure

Runtime reads revealed four isomorphic entries `(0,0)/(0,1)/(1,0)/(1,1)`. The fast morph with stick input, `001C0400`, used its own `(1,1)` entry and bypassed the old no-stick fast-entry hook completely.

The Action logger independently captured `001C0400` for the stick-input fast route and `00040400 → 001B0400` for sword X into the stick-input morph. A motion logger then supplied a controlled structural comparison: under the installed four-hook patch, the already patched no-stick fast route recorded `592 → 583`, while the still-unpatched stick-input fast route produced zero entries at the existing native wrapper under the same logger installation. The zero result therefore demonstrated bypass rather than logger failure.

Special thanks to YouTube user Hazerou. His MH4U USA Gateshark material was one of the important external leads that helped the project get started; it also contained the `001C/001B` fast/morph IDs used when the left stick is moved. That reference guided comparison but was not treated as executable-address evidence; the MH4G runtime logger subsequently established the identities independently.

Adding the fifth hook at `00CA830C` preserved the longer forward-moving fast animation while producing stable GP and red-shield bursts. Three consecutive stick-input fast-morph GPs, stick-input morph isolation, primary follow-ups, and state recovery passed. The final ExeFS v3 then passed clean Azahar automatic loading and extended play with CPU JIT enabled.

The primary explanation for the old stick-related failures changed from “the fast move naturally lacks an opening GP” to “the stick-input fast branch was not covered.”

The finalized MH4G package uses Title ID `000400000011D700`: `MH4G_JPN_v1.2_CB_Fast_Morph_GP_v3_Azahar.zip`, SHA-256 `3D5B864D160497167D2CBD2A3BB6F33128A20A9A6E57CD3940C83387A5BDA941`. Its `code.ips` is 698 bytes / 6 records, SHA-256 `3EB88248D44A9EFE4A83A372A5EA682779BAB2BE8F3E6E8F9101763B88ACA8F4`.

## 2026-08-09: USA no-stick-input v2 proves the regional porting method

Once the MH4G mechanism stabilized, the project tested the porting method rather than copying addresses. A verified 13,627,396-byte USA runtime export was searched offline for full signatures, decoded branches, action-entry structure, and a safe unreferenced cave. This produced a separately relocated four-hook, no-stick-input ExeFS v2 followed by automatic-load and gameplay acceptance. It established that behavior could be transferred while every executable address, external branch, absolute state pointer, and IPS record remained build-specific.

## 2026-08-10: Closing the window question

The project briefly planned to trim the late end of the fast GP window to a chosen frame. The investigation discovered an important methodological error: jumping directly from initialized frame 0 to a target frame skips natural motion/animation events and can preserve an already-open GP state artificially. Late positive samples produced by that method were withdrawn as natural-window evidence.

Natural-progress runtime capture and contact classification then established a conservative boundary: the latest trustworthy positive was frame `27.0`; the earliest classified negative was approximately `30.006`, independently repeated at approximately `30.008`; frames 28–29 remained unresolved. The existing morph reference was positive at `32.5` and negative at `33.0`. The user also compared the result against the official MHGU fast morph with stick input. The final findings were:

- the official MHGU fast morph with stick input also has a GP;
- its late-window feel is close to the current MH4G v3 behavior;
- the current MH4G v3 fast GP window is in practice shorter than the morph window.

The manual trimming task was therefore cancelled. It is confirmed that the fast window does not simply inherit the entire morph window. Natural termination through lifecycle, phase, or motion change is a strong interpretation, but the exact close condition remains unidentified.

The earlier suspicion of a separate inherent GP gap at fast-morph startup was also downgraded. The uncovered stick-input branch explains the old repeatable failure much better; the project did not prove a fixed native startup gap.

## 2026-08-10: USA v3 adds the previously uncovered stick-input branch

The four adjacent USA entry stubs then exposed the stick-input fast entry. The fifth hook was independently encoded as `00CC33B4=EA04A574 → 00DEC98C`; it reused the relocated 640-byte behavior but did not reuse the MH4G IPS record or address. A clean-boot ExeFS RC1 was applied before CPU JIT compiled the guest blocks.

The approximately 22-minute mixed regression covered no-stick/stick-input and morph/fast-morph switching, two consecutive stick-input fast GPs, three consecutive stick-input morph GPs, red-shield bursts, discharge, axe slam/roundslash, rolling, area transitions, and combat recovery. The final five state words were zero. RC1 was promoted byte-for-byte to formal ExeFS v3 for Title ID `0004000000126300`, archive `MH4U_USA_v1.1_CB_Fast_Morph_GP_v3_Azahar.zip`:

- archive SHA-256: `B8C9D2B9F48E0F277BBBB5E2449E8EC110F8728A8AA6DF44B58AEF3B72F7B787`;
- `code.ips`: 698 bytes / 6 records, SHA-256 `683B2AD2A378CA404CA7976F6D3E6721397A77FAB3357AB2C019CEFB5ED932FE`.

## 2026-08-10 to 08-11: EUR rejected same-address reuse and completed independently

The Hazerou EUR Gateshark reference further supported the same gameplay-level Action ID combinations and supplied the EUR player root, but it could not locate executable hooks. A bounded read-only probe first proved that EUR did **not** share the USA hook addresses. The project therefore exported and verified its own 13,627,396-byte EUR runtime image, mapped five hooks and external targets independently, and selected a separately validated zero cave.

The first installer readback occurred at the title screen rather than with an idle hunter in a quest. It was retained only as installation-format evidence and explicitly rejected as execution or safety evidence. Testing restarted from a clean quest-map process: the CPU-JIT-off GDB candidate passed both stick-input branches, GP, bursts, follow-ups, consecutive use, and state recovery, then restored the original hooks and zero cave safely. A clean automatic-load ExeFS RC1 subsequently passed an approximately 10-minute CPU-JIT-on mixed regression and ended with all five state words zero.

The tested RC1 IPS was promoted unchanged to formal EUR ExeFS v3 for Title ID `0004000000126100`, archive `MH4U_EUR_v1.1_CB_Fast_Morph_GP_v3_Azahar.zip`:

- archive SHA-256: `5ECF2013568EA64C133DFCA7374FDDD580C67A869C388265719629DCFC4EB39B`;
- `code.ips`: 698 bytes / 6 records, SHA-256 `56B266F5FA86346D79339EE84258FC878B23B49408684B7B6DF3237AB3024AB2`.

## 2026-08-11: Formal research closure

On 2026-08-11, the project confirmed three finalized releases covering both the no-stick and stick-input branches: MH4G Japanese/localized v1.2, MH4U USA v1.1, and MH4U EUR v1.1. The generic `0x592` GP mechanism and other-weapon transplantation moved to open questions rather than remaining release gaps.

The research objective is closed. Exact `0x592`/Guard causality and other-weapon GP transplantation remain future research questions, not unfinished requirements for these releases. The next stage is publication engineering: final regression as needed, comparison video, release packaging, and bilingual public documentation. The frozen `current_state` files remain research-site archives and are not substitutes for a public README.

## Major failed routes and what they taught

| Route | Result | Lasting conclusion |
| --- | --- | --- |
| Conventional breakpoint/watchpoint tracing | Remote `E01`, failed continuation, or repeated same-value traps | Use scripted hooks, bounded loggers, readback, and behavioral controls as the primary evidence path |
| Copy a single state field or flag | Partial defense without burst, no effect, hit reaction, or broken pose | No single copied bit recreated the complete native GP path |
| Replace Action or handler directly | Animation changed, target behavior incomplete | Action identity, initialization, and external systems are coupled |
| Modify Guard results or event scripts | Could not create the missing window | The result layer is not the GP root cause |
| Delete or submit phases early | Double animation, missing transition, broken follow-ups, or crash | Phases carry both presentation and gameplay logic |
| Globally alias `0x592` to fast resources | GP worked but the morph animation was contaminated | Correct causal direction, wrong scope |
| Clear the marker too early | Fast move degraded into the morph animation | Cleanup must recognize valid motion prefixes |
| Jump directly to a target frame | Natural events were skipped and positives became invalid | Window measurement must allow natural progression |
| Assume regional executable layouts match | EUR differed at the USA reference addresses | Shared gameplay identities do not authorize address or IPS reuse |
| Accept title-screen installer readback as runtime proof | No hunter action could execute there | Installation evidence and gameplay/safety evidence must be recorded separately |

These failures are among the most reusable project outputs. They redirect future work away from a mythical single GP function and toward layered validation of entries, motion, resource lifetime, and the Guard system.
