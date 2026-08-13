# MH4G / MH4U Charge Blade Reverse Engineering and Fast Morph GP

This repository collects **MH4G / MH4U** Charge Blade reverse-engineering notes, research methods, and ExeFS patches for fast-morph Guard Points. The mechanism was discovered and the first complete five-hook implementation was developed against the **localized MH4G v1.2 build**. Concrete MH4G addresses, machine-code values, and mechanism evidence come from that baseline; the MH4U USA/EUR addresses, relocated overlays, runtime evidence, Title IDs, and formal artifacts come from independent work on their respective v1.1 builds.

The layered logic, experimental process, motion/resource-lifetime model, and five-hook porting workflow can also be applied to **MH4U**. This means the method and implementation structure are portable; it does not mean that MH4G addresses or its `code.ips` can be used directly with MH4U. Each MH4U region and revision still requires independent address mapping, overlay relocation, dynamic validation, and clean ExeFS acceptance.

The core mechanism research was formally closed on **2026-08-11**. Finalized ExeFS v3 releases covering fast morphs performed both without moving the left stick and while moving the left stick now exist for localized/Japanese MH4G v1.2, MH4U USA v1.1, and MH4U EUR v1.1.

[中文说明](README.zh-CN.md) · [Development Methodology & Contributions](docs/DEVELOPMENT_METHODOLOGY.md) · [Technical Analysis](docs/TECHNICAL_ANALYSIS.md) · [Research History](docs/RESEARCH_HISTORY.md) · [Porting Notes](docs/PORTING_NOTES.md) · [Release Notes](RELEASE_NOTES.md)

## Research and reproduction environment

The project became practically viable as a sustained dynamic reverse-engineering effort because **Azahar 2126.0** provided a usable and sufficiently stable GDB connection. Earlier versions could not maintain a reliable remote-debugging session, making runtime capture, machine-code patch installation and readback, state inspection, and safe recovery too unstable for a reproducible workflow.

Even on 2126.0, conventional interactive breakpoints and watchpoints were not dependable enough to serve as the primary research workflow. Breakpoint hits, continued execution, and watchpoint traps could produce remote errors, same-value-write noise, or an unusable session. Most runtime evidence was therefore collected through purpose-built GDB scripts, temporary code hooks, low-noise bounded loggers, memory snapshots/exports, and word-for-word readback checks rather than manual breakpoint stepping. A small number of narrow breakpoint or watchpoint trials were used only for controlled diagnosis, and their results required confirmation through scripted records and in-game behavior.

Azahar 2126.0 is the project's **verified research/debugging baseline**. This does not automatically make 2126.0 the minimum version required by players to run the final ExeFS `code.ips`: the release does not depend on GDB, and pure runtime compatibility with earlier Azahar builds has not been systematically tested.

## Features

- Adds GP behavior to the fast morph slash with no stick input.
- Adds the same behavior to the fast morph slash with stick input.
- Keeps no-stick/stick-input morph and fast-morph animations isolated.
- Preserves red-shield phial bursts on successful GP contact.
- Preserves primary follow-ups including axe slam, roundslash, and discharge.
- Passes consecutive stick-input fast GP, re-entry, and state-recovery tests.
- Passes clean ExeFS automatic loading and extended play with CPU JIT enabled.
- Requires no GDB during normal use; the release is loaded through `code.ips`.

## Compatibility

| Game build | Authoritative status | Notes |
| --- | --- | --- |
| MH4G Japanese/localized v1.2 | **ExeFS v3, both input branches covered** | Finalized; five hooks and a 640-byte overlay |
| MH4U USA v1.1 | **ExeFS v3, both input branches covered** | Automatic loading, CPU-JIT-on operation, and an approximately 22-minute mixed gameplay regression passed |
| MH4U EUR v1.1 | **ExeFS v3, both input branches covered** | Automatic loading, CPU-JIT-on operation, and an approximately 10-minute mixed gameplay regression passed |

Do not reuse addresses, IPS files, or `code.ips` files across regions or game revisions.

## Installation

1. Close Azahar completely.
2. Back up existing mods and save states.
3. Merge the release archive's `load` folder into the Azahar user directory.
4. Confirm that the final file is in the directory matching the selected release:

   | Release | Final path |
   | --- | --- |
   | MH4G Japanese/localized v1.2 | `load/mods/000400000011D700/exefs/code.ips` |
   | MH4U USA v1.1 | `load/mods/0004000000126300/exefs/code.ips` |
   | MH4U EUR v1.1 | `load/mods/0004000000126100/exefs/code.ips` |

5. Restart Azahar and enter the game from a normal in-game save.

If another `code.ips` already exists at that location, do not overwrite it or place both files side by side. Back it up and merge the IPS records correctly.

## Uninstallation

Close Azahar completely, remove or rename the `code.ips` in the selected release's Title-ID directory, and restart the game. The mod does not alter the ROM or normal in-game save data.

## Important limitations

- Fully restart the game after enabling, disabling, or replacing the ExeFS patch.
- Do not load an Azahar save state created under a different mod state; a save state can restore old code memory.
- Do not combine this ExeFS build with the project's GDB or Gateshark installers.
- Guard direction and input timing still matter in combat. Being hit does not by itself prove a patch failure.
- This is an unofficial research mod and is not affiliated with CAPCOM. A legally obtained game environment is required.

## Verified behavior

The final v3 build has closed-loop validation for:

- automatic loading of all five hooks and the 640-byte overlay;
- single, naturally ending fast animations with no stick input and with stick input;
- isolation between fast-morph and morph animations;
- fast-morph GP with no stick input and with stick input, morph GP regression, and red-shield phial bursts;
- consecutive stick-input fast-morph GP without degrading into the morph animation;
- primary follow-ups including axe slam, roundslash, and discharge;
- rolling, area transitions, sheathing, combat recovery, and state cleanup;
- extended real play with CPU JIT enabled.

A later comparison against the official MHGU implementation confirmed that its fast morph with stick input also has a GP and that its late-window feel is close to the current MH4G v3 behavior. The v3 fast GP window was further observed to be shorter than the morph window, so the former plan to trim its tail manually has been cancelled.

What is confirmed is that the fast window does not simply inherit the entire morph window. A strong interpretation is that native action lifecycle, phase, or motion changes end the GP naturally. The exact close condition has not been located, so no single Action or flag should be described as proven to control it.

## Release identity and checksums

The three archives were repackaged on **2026-08-12** to replace older neutral/directional wording with clearer left-Circle-Pad descriptions. This was a documentation-only package refresh: the filenames, `code.ips` files, overlays, hooks, and tested gameplay behavior did not change. The ZIP hashes below identify the current downloadable archives.

### MH4G v1.2 localized/Japanese — ExeFS v3, both no-stick and stick-input branches covered

- Title ID: `000400000011D700`
- Release archive: `MH4G_JPN_v1.2_CB_Fast_Morph_GP_v3_Azahar.zip`
- ZIP SHA-256: `60616F01515BF84BE8FCCB8206AA6123467B8C48F251BE9CA4625002FC0ACCBC`
- `code.ips`: 698 bytes, 6 records
- `code.ips` SHA-256: `3EB88248D44A9EFE4A83A372A5EA682779BAB2BE8F3E6E8F9101763B88ACA8F4`
- 640-byte overlay SHA-256: `E82E27E04C7163BFBEACBD5ED5B02115B7DFC814803A4EB326102A7B5DC25D03`

### MH4U USA v1.1 — ExeFS v3, both no-stick and stick-input branches covered

- Title ID: `0004000000126300`
- Release archive: `MH4U_USA_v1.1_CB_Fast_Morph_GP_v3_Azahar.zip`
- ZIP SHA-256: `C0058F7072850B2E5007324554B0AEA8C404B0CB3E40CD17A6B20B6B3CBCF303`
- `code.ips`: 698 bytes, 6 records
- `code.ips` SHA-256: `683B2AD2A378CA404CA7976F6D3E6721397A77FAB3357AB2C019CEFB5ED932FE`
- 640-byte overlay SHA-256: `E529D92B9ECFD8BE21D084A87250EC426DF0C1091C0F488AFF72B145783E1F0A`
- Stick-input fifth hook: `00CC33B4=EA04A574 -> 00DEC98C`
- Clean automatic loading, CPU-JIT-on operation, and an approximately 22-minute mixed gameplay regression passed; all five final state words were zero.

### MH4U EUR v1.1 — ExeFS v3, both no-stick and stick-input branches covered

- Title ID: `0004000000126100`
- Release archive: `MH4U_EUR_v1.1_CB_Fast_Morph_GP_v3_Azahar.zip`
- ZIP SHA-256: `D6350D54B3CF27804575DB46CFF770580FC1438240887DF2786CB0F3A63E1793`
- `code.ips`: 698 bytes, 6 records
- `code.ips` SHA-256: `56B266F5FA86346D79339EE84258FC878B23B49408684B7B6DF3237AB3024AB2`
- 640-byte overlay SHA-256: `FB318D5158E4028C45F5FB173D32D9FC5E46D9E179E0FD521D257FAA13949853`
- The formal IPS is byte-identical to the dynamically tested RC1 candidate; deterministic double-build and in-archive file validation passed.
- Clean automatic loading, CPU-JIT-on operation, and an approximately 10-minute mixed gameplay regression passed.

## Documentation boundary

`archive/current_state_research_archive.md` is a frozen chronological field archive. It intentionally preserves provisional interpretations, failed routes, later corrections, and a prepended closure chapter written before the USA/EUR v3 release work was fully reflected there. The closure chapter remains authoritative for the MH4G main-research decision at its freeze point, but its old USA/EUR status is superseded by this README, `RELEASE_NOTES.md`, and the edited documents under `docs/`. Do not use the raw archive directly as the public README.

## Development methodology and contributions

This project combined AI-produced reverse-engineering analysis and implementation with human-operated runtime experiments. ChatGPT/Codex produced the project-authored analysis, experiment designs, scripts, ARM hook/overlay implementation, builders, validators, and documentation drafts. The project author operated Azahar/GDB, collected all live data, executed the gameplay comparisons, classified samples, and performed final runtime acceptance. See [Development Methodology and Contribution Disclosure](docs/DEVELOPMENT_METHODOLOGY.md) for the full boundary.

## Acknowledgements

Special thanks to YouTube user [Hazerou](https://www.youtube.com/@hazerou8601). His two-part MH4U Charge Blade cheat-code material provided important leads that helped this project get started. It also later helped identify that fast and morph actions use separate branches depending on whether the left stick is left neutral or moved. The project subsequently recorded and validated those Action IDs and entry structures independently in the running MH4G Japanese/localized v1.2 build.
