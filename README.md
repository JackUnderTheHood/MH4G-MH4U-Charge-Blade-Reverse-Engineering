# MH4G / MH4U Charge Blade GP Reverse Engineering

[中文说明](README.zh-CN.md) · [Download releases](https://github.com/JackUnderTheHood/MH4G-MH4U-Charge-Blade-Reverse-Engineering-/releases) · [Technical analysis](docs/TECHNICAL_ANALYSIS.md) · [Checksums](ARTIFACTS.md)

This project documents the reverse engineering of Charge Blade Guard Points and red-shield phial bursts in *Monster Hunter 4G / 4 Ultimate*. It also provides two sets of runtime-tested Azahar ExeFS patches.

The project began with one goal: give fast morph slash a Guard Point. The investigation eventually separated four mechanisms that are easy to confuse:

1. when during an action the GP can actually block an attack;
2. whether the guard is actually accepted, rather than merely showing guard sparks;
3. how GP guard performance changes recoil;
4. how a successful red-shield GP reaches the phial-burst collision and damage pipeline.

The final v4 implementation no longer borrows another action's logical identity as v3 did. It preserves the native fast-morph actions and animations, extends an existing native guard-timing call, and enters the native phial-burst path only after a successful red-shield contact.

## Mechanism at a glance

![Guard Point and phial-burst mechanism flowchart](assets/gp-phial-burst-flowchart.svg)

The diagram separates three mechanisms that are often mistaken for one switch:

- `state+F0 bit 0x10` is a required condition for the attack to be genuinely blocked. Guard sparks alone do not prove that the attack was stopped.
- `state+F0 bit 0x2000` does not make a GP guard check valid by itself. Once guarding is valid, it reduces the internal recoil-classification value by 10, with a minimum of 1.
- A successful contact does not automatically create a phial burst. Without red shield, the result is guard only; with red shield, the validated path continues through the native phial-burst object, collision processing, result queue, and HP damage.

The image is a readable overview rather than a substitute for the evidence chain. Exact call sites, Action filters, and the different fast-morph and charge-slash burst paths are documented in [Technical Analysis](docs/TECHNICAL_ANALYSIS.md).

## Which package should I install?

Choose **one** of the following packages.

### Fast Morph GP v4

For players who only want fast morph slash to gain a GP.

- Fast morphs gain a GP both without moving the left stick and while moving it.
- A successful GP without red shield does not create a phial burst.
- A successful GP with red shield creates one phial burst.
- AED and axe-slam follow-ups remain available.
- Sword-mode charge-slash GP is not included.

### Double Charge Slash GP v1 (combined package)

For players who want both features.

- It **already includes Fast Morph GP v4**.
- Sword mode gains a GP while the shield is visibly held forward during charging.
- Small and medium recoil do not interrupt the charge.
- Large recoil still interrupts the charge according to the original rules.
- A phial burst occurs only after a successful GP while red shield is active.

Do not install both packages. The combined package already contains Fast Morph GP v4, so installing the standalone patch as well would create conflicting `code.ips` records.

## Supported builds

| Build | Title ID | Fast Morph GP v4 | Combined v1 |
|---|---|---:|---:|
| MH4G Japanese/localized v1.2 | `000400000011D700` | Tested | Tested |
| MH4U USA v1.1 | `0004000000126300` | Tested | Tested |
| MH4U EUR v1.1 | `0004000000126100` | Tested | Tested |

“Supported” means that each build received independent address mapping, patch construction, runtime installation, gameplay testing, and safe restoration. Addresses differ by region; changing only the folder name or Title ID does not port a patch.

## Installation

1. Fully close Azahar.
2. Download the archive that exactly matches the game region and revision.
3. Merge the archive's `load` folder into the Azahar user directory.
4. Confirm that the final file is located at:

   ```text
   load/mods/<Title ID>/exefs/code.ips
   ```

5. Restart the game and load a normal in-game save.

The default Azahar user directory on Windows is usually:

```text
C:\Users\<username>\AppData\Roaming\Azahar
```

## Important notes

- Only one final `code.ips` can be active for a given Title ID.
- If another ExeFS patch is installed, back it up and merge IPS records correctly. Do not assume that overwriting one file combines two patches.
- Do not enable old releases, GDB test patches, or experimental Gateshark codes at the same time.
- Fully close Azahar before enabling, disabling, or replacing the patch.
- Do not load save states created across different patch states; a save state may restore stale code memory.
- The patch does not modify the ROM or normal game saves.

To uninstall, fully close Azahar, move or rename `code.ips`, and restart the game.

## Validated behavior

- Both fast-morph input branches keep their native animations and play once.
- GP and phial-burst behavior is correctly separated by red-shield state.
- Every tested red-shield GP produced only one burst.
- AED and axe-slam follow-ups work after fast-morph GP.
- Regular morphs, native morph GPs, and held-R guarding remain unaffected.
- Short hold, normal release, and automatic full hold remain normal for charge slash.
- Small and medium charge-GP recoil continue charging; large recoil interrupts it.
- All three regions passed installation readback, state checks, and safe restoration.

Multi-hit monster attacks were not covered by a dedicated test matrix. No repeated burst was observed in existing testing, but that is not an exhaustive proof for every multi-hit attack.

## What changed from v3?

V3 used five hooks and a 640-byte overlay. It let the fast action borrow the logical identity of an existing GP action while preserving the fast animation. This proved the feature and exposed the important mechanism differences, but it also required complex identity and resource-lifetime management.

V4 uses the later mechanism findings:

- no motion/action identity replacement;
- no five-hook resource overlay;
- two hooks and a 264-byte cave for standalone Fast Morph GP v4;
- three hooks and a 792-byte cave for the combined package;
- GP, recoil, and phial burst each reconnect to a verified native path.

V4 is therefore a different runtime implementation, not merely a documentation or version-number update.

The cause-and-effect sequence is:

```text
v3 completes the first usable five-hook solution
→ v3 becomes the stable A baseline for controlled comparisons
→ A/B experiments separate guard acceptance, recoil adjustment, and burst delivery
→ native mask-2 timing is found in the fast action itself
→ final v4 replaces identity borrowing with a two-hook native-timing implementation
→ charge-slash GP reuses the separated mechanisms as an independent cross-check
```

V3 was therefore not a discarded mistake. It was the functional baseline that made the more precise v4 mechanism possible. The withdrawn “v4” repack and the final real v4 are distinguished explicitly in [Research History](docs/RESEARCH_HISTORY.md).

## Documentation

- [Technical Analysis](docs/TECHNICAL_ANALYSIS.md)
- [Research History](docs/RESEARCH_HISTORY.md)
- [Porting Notes](docs/PORTING_NOTES.md)
- [Development Methodology and Contributions](docs/DEVELOPMENT_METHODOLOGY.md)
- [Release Artifacts and Checksums](ARTIFACTS.md)
- [Release Notes](RELEASE_NOTES.md)
- [Documentation Rewrite Summary](DOCUMENTATION_REWRITE_SUMMARY.md)
- [Research Archive Guide](archive/README.md)

## Research environment

Azahar 2126.0 was the validated dynamic-research baseline. Earlier builds could not maintain a sufficiently stable GDB connection. Even on 2126.0, conventional breakpoints and watchpoints were not reliable enough for primary data collection, so the project relied on automated GDB scripts, temporary code hooks, bounded loggers, memory exports, and word-for-word readback.

The final ExeFS patches do not require GDB. A validated research baseline should not be interpreted as a proven minimum Azahar version for ordinary play.

## Contributions and acknowledgement

ChatGPT/Codex produced the project-authored static analysis, experiment designs, GDB scripts, ARM hooks and caves, builders, validators, regional relocations, and documentation drafts. JackUnderTheHood operated Azahar/GDB, collected runtime data, performed gameplay comparisons, classified samples, identified visible failures, and completed final runtime acceptance.

Special thanks to Hazerou. Publicly shared MH4U Charge Blade Gateshark material provided an important early lead and helped draw attention to the separate action branches used when the left stick is not moved and when it is moved. This project later recorded the action identities, located each regional implementation, and tested every supported build independently. No Hazerou code or assets are included in the releases.

## Document boundary

The public documents summarize verified findings and clearly stated remaining limits. The original `current_state.md` is a research-site archive larger than one million bytes. It contains candidate hypotheses, failed experiments, later-retracted interpretations, and historical next-step notes, so it should not be used directly as a README or final technical explanation.
