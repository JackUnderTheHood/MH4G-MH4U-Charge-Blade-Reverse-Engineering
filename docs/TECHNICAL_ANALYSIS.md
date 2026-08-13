# Technical Analysis: MH4G / MH4U Charge Blade Fast Morph GP

This document describes the verified structure of the final ExeFS v3 build, the boundary between evidence and interpretation, and the questions that remain open. It is a public-facing synthesis of the research archive rather than a replay of every failed experiment.

The division between AI-produced reverse-engineering work and human-operated runtime validation is documented in [Development Methodology and Contribution Disclosure](DEVELOPMENT_METHODOLOGY.md).

Scope: the mechanism reconstruction, motion sequences, and detailed v6 state-machine explanation were established on the **localized MH4G v1.2 build**. Sections 4 and 6 distinguish those results from the independently relocated and dynamically accepted MH4U USA/EUR v1.1 builds. Gameplay-level action identities are shared across the three targets, but executable addresses, branch encodings, code caves, overlays, and IPS records are build-specific and are not interchangeable.

## 1. The problem was not a standalone “GP switch”

The player sees one fast morph slash, but the implementation spans several layers:

```text
input and stick state
→ action-entry parameters
→ Action routing
→ motion / phase submission
→ resource objects and lifetime
→ Guard / collision result
→ red-shield phial burst and follow-ups
```

Extensive early tests of individual fields, result scripts, collision records, and generic flags did not reveal an independently portable GP switch. The working solution combines morph logic, fast visual resources, and reliable lifetime cleanup.

### 1.1 Azahar 2126.0 made the dynamic method viable

The conclusions do not come from static disassembly alone. The decisive causal evidence required action capture, bounded logging, machine-code installation, word-for-word readback, behavior testing, state reinspection, and safe restoration within controlled runtime sessions. That workflow depends on a GDB connection that remains readable and writable across repeated pause/continue cycles.

Earlier Azahar versions could not maintain a sufficiently stable GDB session. Under those conditions, an observed patch result could not be separated reliably from incomplete installation, failure to execute new code, JIT caching, or a remote-connection fault, and failed experiments could not always be restored safely. **Azahar 2126.0 was therefore the verified debugging baseline that made the project viable as a reproducible dynamic-research effort**, not merely an incidental emulator version.

“Usable GDB transport” must be distinguished from “reliable conventional breakpoint debugging.” Azahar 2126.0 solved the basic connection and guest-memory access problem, but interactive breakpoints and watchpoints were still unsuitable as the main evidence-collection method. Breakpoints could be accompanied by remote errors such as `E01`, and continuing could leave the session unusable. Watchpoints could trap on same-address, same-value writes and stop repeatedly during high-frequency state updates. A single stopped frame was therefore not sufficient evidence of an action path or GP boundary.

The final workflow used **scripted instrumentation and recoverable experiments**:

- GDB preflight/enable/status/disable scripts checked the baseline, installed machine code, performed word-for-word readback, inspected state, and restored safely;
- temporary code hooks and low-noise bounded loggers wrote Action, motion ID, cursor, source, and prefix sequences into allocated log areas;
- memory snapshots, chunked code exports, and offline SQLite/disassembly analysis supported broad searches and strict comparisons;
- successful guarding, red-shield phial bursts, animation identity, follow-ups, and state recovery provided final in-game causal acceptance.

A few narrow breakpoints at fixed code points or literal-address watchpoints were tried to validate a diagnostic tool or reduce a candidate set, but they were not the sustainable primary capture route. Every major conclusion ultimately required agreement between scripted records, machine-code readback, and in-game behavior.

This is a research-reproduction requirement. The final v3 release loads automatically through ExeFS and does not require GDB, so the available evidence does not establish 2126.0 as the minimum player-facing runtime version.

## 2. Confirmed action identities

| Move | Action |
| --- | --- |
| Fast morph with no stick input | `000B0400` |
| Morph with no stick input | `00060400` |
| Fast morph with stick input | `001C0400` |
| Morph with stick input | `001B0400` |

MH4G v1.2 has four isomorphic entries at `00CA82D4..00CA8313`. Before entering the shared body at `00CAC2D4`, two parameter axes select no-stick/stick input and morph/fast-morph behavior:

| Entry | `(r2,r1)` | Meaning |
| --- | --- | --- |
| `00CA82D4` | `(0,0)` | no-stick morph |
| `00CA82E4` | `(0,1)` | no-stick fast |
| `00CA82F4` | `(1,0)` | stick-input morph |
| `00CA8304` | `(1,1)` | stick-input fast |

The older build hooked only the no-stick fast entry. Moving the stick selected the uncovered `(1,1)` branch, which is the primary structural explanation for the earlier reports of intermittent fast-GP failure.

This was verified with two bounded runtime loggers. The Action logger captured `001C0400` for a stick-input fast morph and `00040400 → 001B0400` for the sword-X-to-stick-input-morph route. Under the same installed motion logger and four-hook patch, the already patched no-stick fast control produced `592 → 583`, while the still-unpatched stick-input fast route produced no entries at the existing native wrapper. The zero result therefore had a positive control and showed that the branch bypassed the patched submission path; it was not treated as random logger failure.

## 3. `0x592` and `0x583`

Low-noise runtime logging for the no-stick-input actions produced:

```text
morph:      592 → 583
native fast:     583
```

This identifies `0x592` as a critical observable difference between the morph logic and the native fast route, but it should not be named a “GP function” or “GP flag.” Reconstructing the morph-associated `0x592` motion/resource lifecycle while retaining the fast visual resources was sufficient to make this patch produce GP behavior and red-shield phial bursts through the existing game systems.

That result does **not** isolate `0x592` itself as the causal GP switch. The unlocated consumer could depend on a resource reached through that context, a phase/event transition, or a downstream Guard/collision condition. The project therefore establishes a verified relationship sufficient for this Charge Blade patch, not a universal GP meaning for `0x592` in other moves or weapons.

## 4. Final patch structure

The formal localized MH4G v1.2 build contains five hooks and one 640-byte overlay:

| Role | Address | Original | Patched |
| --- | --- | --- | --- |
| Resource overlay | `00941334` | `E92D5FF0` | `EA12326B` |
| Native motion wrapper | `00B0D0A0` | `EBFFA103` | `EB0B0340` |
| No-stick fast entry | `00CA82EC` | `E3A01001` | `EA0496BC` |
| Stick-input fast entry | `00CA830C` | `E1A01002` | `EA0496B4` |
| Action-finish wrapper | `00CAC478` | `EBF96833` | `EB048661` |

The runtime overlay occupies `00DCDCE8..00DCDF67`, or 640 bytes / 160 words. v3 adds only the stick-input fast-entry hook; the overlay itself is byte-identical to the v6 machine code that completed the full dynamic regression.

At a high level:

```text
no-stick or stick-input fast entry
→ clear stale state and arm a fast marker
→ preserve the stick-input axis while entering the corresponding morph logic
→ establish a temporary fast visual-resource context during motion submission
→ let the existing Guard/collision system create GP and the red-shield system add the burst
→ restore resources and internal state through native-motion prefixes and the action boundary
```

The patch does not permanently replace the fast-morph Action with the morph Action, nor does it globally replace morph resources. It combines logic and visual resources for a bounded lifetime and then restores them.

### 4.1 Independent MH4U relocations

The USA and EUR releases reuse the validated behavior and 640-byte layout, but each executable was mapped independently from its own runtime image. Their hook words and external branches were re-encoded for the target addresses; neither release is a renamed or Title-ID-only copy of the MH4G IPS.

| Target | Title ID | Five runtime hooks | Overlay range |
| --- | --- | --- | --- |
| MH4G Japanese/localized v1.2 | `000400000011D700` | `00941334`, `00B0D0A0`, `00CA82EC`, `00CA830C`, `00CAC478` | `00DCDCE8..00DCDF67` |
| MH4U USA v1.1 | `0004000000126300` | `00957C4C`, `00B242B8`, `00CC3394`, `00CC33B4`, `00CC7520` | `00DEC890..00DECB0F` |
| MH4U EUR v1.1 | `0004000000126100` | `00957C84`, `00B24308`, `00CC33E4`, `00CC3404`, `00CC7570` | `00DEC890..00DECB0F` |

The installed USA hook words are `EA12530F`, `EB0B21A4`, `EA04A57C`, `EA04A574`, and `EB049521`. The installed EUR hook words are `EA125301`, `EB0B2190`, `EA04A568`, `EA04A560`, and `EB04950D`. In both builds, the two fast-entry hooks converge on the relocated shared wrapper at `00DEC98C` while preserving the no-stick/stick-input selector.

| Target | 640-byte overlay SHA-256 | Formal `code.ips` SHA-256 |
| --- | --- | --- |
| MH4G Japanese/localized v1.2 | `E82E27E04C7163BFBEACBD5ED5B02115B7DFC814803A4EB326102A7B5DC25D03` | `3EB88248D44A9EFE4A83A372A5EA682779BAB2BE8F3E6E8F9101763B88ACA8F4` |
| MH4U USA v1.1 | `E529D92B9ECFD8BE21D084A87250EC426DF0C1091C0F488AFF72B145783E1F0A` | `683B2AD2A378CA404CA7976F6D3E6721397A77FAB3357AB2C019CEFB5ED932FE` |
| MH4U EUR v1.1 | `FB318D5158E4028C45F5FB173D32D9FC5E46D9E179E0FD521D257FAA13949853` | `56B266F5FA86346D79339EE84258FC878B23B49408684B7B6DF3237AB3024AB2` |

All three formal IPS files are 698 bytes with six records: five 4-byte hooks and one 640-byte overlay record.

## 5. Why the v6 prefix state machine was necessary

Earlier overlays could produce the fast animation and GP, but repeated use exposed stale state and morph-animation contamination. The important observed sequences were:

```text
A: standalone fast    592 → 583
B: fast GP follow-up  592 → 592 → 583 → 579 → 56B
C: later morph        567 → 592
```

Before an actual resource switch exists, v6 temporarily reuses the fifth state word as a prefix stage:

- `592` preserves the marker and records the prefix stage;
- `583` is allowed to continue when preceded by `592`;
- any other motion, or `583` without a preceding `592`, restores the five state words before forwarding;
- after the overlay is established, the fifth word resumes its real saved-length role and no longer overlaps the prefix stage.

This permits repeated fast chains, while the morph's `567` prefix clears a dormant marker before the morph `592`, preventing fast-resource contamination.

## 6. Validation scope

The final result is supported by both offline and dynamic validation:

- reverse parsing of the IPS, record offsets, little-endian payloads, and branch targets;
- overlay size, key signatures, and SHA-256;
- clean Azahar automatic loading after a full restart rather than GDB injection;
- readback of all five hooks, the overlay, `592/583` literals, and state words;
- no-stick/stick-input and morph/fast-morph animation isolation;
- GP, red-shield phial bursts, and consecutive stick-input fast GP;
- primary follow-ups including axe slam, roundslash, and discharge;
- action completion, re-entry, rolling, area transitions, and state recovery;
- extended gameplay with CPU JIT enabled.

The internal state may be fully zero or, in specific tested cases, contain a safe dormant marker that the next valid entry recovers. A dormant `1,0,0,0,0` state was explicitly tested against a later morph and did not contaminate its animation; later accepted sessions also ended with all five words at zero.

Regional acceptance was not inferred from the MH4G result:

| Target | Independent runtime acceptance |
| --- | --- |
| MH4G Japanese/localized v1.2 | Clean ExeFS automatic loading, five-hook readback, no-stick/stick-input isolation, consecutive GP and follow-ups, plus approximately 10 minutes of CPU-JIT-on stick-input combat testing; final accepted status was all zero. |
| MH4U USA v1.1 | Clean ExeFS automatic loading with CPU JIT enabled and an approximately 22-minute mixed regression covering branch switching, consecutive GP, bursts, follow-ups, rolling, area transitions, and recovery; all five final state words were zero. The accepted RC1 IPS was promoted unchanged. |
| MH4U EUR v1.1 | A GDB candidate first passed the core branches with CPU JIT off, was safely restored, and was followed by clean ExeFS automatic loading and an approximately 10-minute CPU-JIT-on mixed regression; all five final state words were zero. The accepted RC1 IPS was promoted unchanged. |

## 7. Final conclusion on the fast GP window

A plan once existed to trim the late end of the fast GP window manually, but that task has been cancelled. Natural no-stick fast-morph measurements placed the latest trustworthy positive at frame `27.0` and the earliest contact-classified negative at approximately `30.006`, independently repeated at approximately `30.008`; frames 28–29 remain unresolved. The existing morph reference was positive at `32.5` and negative at `33.0`, so the fast window is already demonstrably shorter.

The user's later comparison with MHGU also confirmed that the official fast morph with stick input has a GP, and that its late-window feel is close to the current MH4G v3 result. This MHGU comparison is gameplay observation, not a frame-exact cross-game measurement.

Confirmed: the fast window does not simply inherit the entire morph window.

Strong interpretation: native action lifetime, phase, or motion changes end the GP naturally.

Not proven: a specific Action, single flag, or already located fixed condition closes it. The investigation also found that jumping directly from frame 0 to a target frame skips natural events, so late positive results produced by that method are not treated as natural-window evidence.

The earlier suspicion of an inherent GP gap at fast-morph startup is likewise downgraded rather than established. The uncovered stick-input branch explains the old reproducible failures much better: before the fifth hook, stick input bypassed the patched route entirely. Bad facing or input timing can still produce a normal failed guard, but the project did not prove a separate built-in startup gap.

## 8. Evidence levels

### Verified

- the four action identities and the stick-input branch structure;
- the five-hook, 640-byte final machine structure;
- morph/fast-morph visual isolation, GP, bursts, primary follow-ups, and state recovery;
- the uncovered stick-input branch as the main explanation for old failures;
- the fast window not being a complete copy of the morph window.

### Strongly inferred

- GP emerges when the reconstructed `0x592`-related motion/resource context reaches the existing Guard/collision system;
- later lifecycle, phase, or motion changes naturally end the fast window.

### Open questions

- the exact GP close condition and its consumer;
- the complete causal relationship between `0x592` and the generic Guard/collision system;
- whether this can be abstracted into a safe, general GP-porting method for other weapons.

These open questions do not affect the completed status of the MH4G Japanese/localized, MH4U USA, or MH4U EUR v3 releases.
