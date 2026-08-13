# MH4G / MH4U Charge Blade Porting Notes for Regions and Revisions

This document is for researchers porting the finalized MH4G Japanese/localized v1.2 design to another region or revision. It is not a copy-and-paste address list.

## 1. Authoritative version status

| Target | Status |
| --- | --- |
| MH4G Japanese/localized v1.2 | ExeFS v3 covering both no-stick and stick-input branches; main research closed |
| MH4U USA v1.1 | ExeFS v3 covering both no-stick and stick-input branches; automatic loading, CPU-JIT-on operation, and an approximately 22-minute mixed gameplay regression passed |
| MH4U EUR v1.1 | ExeFS v3 covering both no-stick and stick-input branches; automatic loading, CPU-JIT-on operation, and an approximately 10-minute mixed gameplay regression passed |

The formal EUR v3 archive is `MH4U_EUR_v1.1_CB_Fast_Morph_GP_v3_Azahar.zip`, SHA-256 `5ECF2013568EA64C133DFCA7374FDDD580C67A869C388265719629DCFC4EB39B`. Its `code.ips` is 698 bytes / 6 records, SHA-256 `56B266F5FA86346D79339EE84258FC878B23B49408684B7B6DF3237AB3024AB2`; its 640-byte overlay SHA-256 is `FB318D5158E4028C45F5FB173D32D9FC5E46D9E179E0FD521D257FAA13949853`. The formal IPS is byte-identical to the dynamically tested RC1 and passed deterministic double-build and in-archive file validation.

The formal USA v3 archive is `MH4U_USA_v1.1_CB_Fast_Morph_GP_v3_Azahar.zip`, SHA-256 `B8C9D2B9F48E0F277BBBB5E2449E8EC110F8728A8AA6DF44B58AEF3B72F7B787`. Its `code.ips` is 698 bytes / 6 records, SHA-256 `683B2AD2A378CA404CA7976F6D3E6721397A77FAB3357AB2C019CEFB5ED932FE`; its 640-byte overlay SHA-256 is `E529D92B9ECFD8BE21D084A87250EC426DF0C1091C0F488AFF72B145783E1F0A`. The stick-input fifth hook is `00CC33B4=EA04A574 -> 00DEC98C`; an approximately 22-minute CPU-JIT-on mixed regression passed and all five final state words were zero.

The existence of candidate scripts, RC archives, or later experimental artifacts does not automatically promote them to an authoritative release. A build becomes formal only after independent mapping for that version, dynamic validation, automatic-load regression, and explicit confirmation in the current public release-status documents, supported by the appropriate dated archive evidence. The frozen archive's prepended closure section must not be used by itself to downgrade a later completed port.

## 2. Debugging-environment prerequisite

The project's verified research baseline is **Azahar 2126.0**. Earlier versions could not maintain a stable enough GDB connection for sustained dynamic reverse engineering. Before beginning a regional port, verify that the selected emulator build can reliably handle repeated pause/continue cycles, guest-memory reads and writes, bounded code export, word-for-word patch readback, and safe restoration. If it cannot, fix or replace the debugging environment first; do not interpret connection failures as evidence that a target function is absent or a patch has tested negative.

Azahar 2126.0 is the recommended known-working baseline. Another emulator build may substitute only after equivalent stability has been demonstrated. This requirement applies to research and porting and does not automatically define the minimum player-facing version for the final ExeFS mod.

Do not confuse “GDB can connect” with “a port can be completed by conventional breakpoint/watchpoint stepping.” Even on 2126.0, interactive breakpoints and watchpoints remained unreliable auxiliary diagnostics that could produce remote errors, failed continuation, and high-frequency or same-value-write noise. Porting should favor the scripted workflow used by this project: read-only preflight, recoverable enable/status/disable, temporary code hooks, bounded loggers, memory snapshots, chunked exports, and offline branch validation, with runtime behavior and final state checking each other.

## 3. Do not copy MH4G addresses directly

The final JPN v1.2 hooks are:

```text
00941334  resource overlay
00B0D0A0  native motion wrapper
00CA82EC  no-stick fast entry
00CA830C  stick-input fast entry
00CAC478  action finish wrapper
```

These are **JPN v1.2 runtime addresses**. Another region may retain similar Action IDs, function shapes, or relative distances while moving every executable address. In particular, `00CA830C` must never be written directly into USA/EUR.

Porting should use full instruction context, call relationships, and decoded branch targets rather than one four-byte signature. A single ARM word is not unique enough in a large executable.

## 4. Semantic objects that must be relocated

Every target build must independently confirm:

- exact game revision, Title ID, and code load base;
- player-pointer root and Action read path;
- fast-morph/morph Action identities with no stick input and with stick input;
- the four isomorphic morph-entry stubs and their `(direction, fast)` axes;
- resource-submission entry and original prologue;
- native-motion submission callsite;
- action-finish/recovery callsite;
- a sufficiently large safe code cave;
- every external branch and absolute-address literal in the overlay;
- runtime-address to IPS-offset conversion.

The existing research provides cross-version leads for two Action ID combinations: `000B/0006` when the left stick is neutral and `001C/001B` when it is moved. They still require target-build confirmation through runtime recording or equivalent target-build entry mapping plus behavioral isolation. External cheat-code material is not sufficient proof by itself.

## 5. Recommended porting workflow

### Phase A: read-only mapping

1. Fix the target ROM region, game update, Title ID, and emulator version; prefer the verified Azahar 2126.0 debugging baseline and confirm stable scripted memory access/readback.
2. In a quest map, confirm readable runtime code and record the code load base.
3. Export executable code in bounded chunks; probe a safe chunk size first if the remote stub limits large transfers.
4. Search offline for complete function signatures, call relationships, and adjacent-entry structure.
5. Read the four morph-entry stubs and prove the mapping between the two parameter axes and four moves.
6. Scan candidate code caves and prove that the full range is zero, large enough, and has no known references or incoming branches.

Phase A must not write guest memory or produce a release IPS.

### Phase B: overlay relocation

1. Use the validated 640-byte JPN v6 overlay as the behavioral baseline.
2. Preserve the semantics of internal relative branches and the `592/583` literals.
3. Re-encode every external branch to the target build's functions.
4. Update target-specific absolute state or resource addresses.
5. Verify the source, target, link bit, and range of every ARM B/BL.
6. Verify an exact 640-byte image and an in-bounds state area.

Do not apply one fixed address delta to the whole design. Every external target must be justified by the target executable's own instructions and call graph.

### Phase C: offline build and reverse validation

1. Generate the candidate overlay and IPS.
2. Parse the IPS back and compare every record offset, length, and little-endian payload.
3. Generate SHA-256 values for the overlay, `code.ips`, and ZIP.
4. Decode all branches again and verify their targets.
5. Confirm reproducibility: two consecutive builds should match, or any non-reproducible metadata source must be documented.

### Phase D: recoverable dynamic candidate

1. Disable CPU JIT and stop with an idle hunter in a quest map.
2. Run a read-only preflight that requires every original hook and the complete cave baseline to match.
3. Write the overlay first, read back signatures and initial state, then install hooks last.
4. Perform a no-monster fast-animation smoke test first: correct identity, one playback, natural end, no crash.
5. Test morph-animation isolation next.
6. Only then test monster GP, red-shield bursts, follow-ups, and consecutive input.
7. The dynamic candidate needs a safe path that restores original hooks and zeroes the cave.

Phase D is the safest default for a new relocation. For a minimal extension to an already dynamically accepted overlay, a clean-boot ExeFS RC applied before guest blocks are JIT-compiled may replace live hot-writing; USA v3 followed this route for its fifth hook. That route still requires target-build mapping, baseline and installed-code readback, behavioral-isolation tests, a rollback path, and the full Phase E acceptance gate. EUR v3 used the CPU-JIT-off recoverable candidate route before ExeFS acceptance.

### Phase E: ExeFS release acceptance

1. Package the exact dynamically tested machine code as `code.ips`; do not change behavior during packaging.
2. Close the emulator completely before installation.
3. Clean boot from a normal in-game save without a GDB enable script or a save state.
4. Read back hooks, overlay, literals, and initial state to prove automatic ExeFS loading.
5. Complete minimal fast-morph/morph smoke tests, then an extended mixed regression with CPU JIT enabled.
6. End in a known safe state and document the Title ID, revision, hashes, and save-state restriction.

## 6. Minimum regression matrix

A port covering both the no-stick and stick-input branches should pass at least:

| Category | Required checks |
| --- | --- |
| Animation | No-stick fast morph, no-stick morph, stick-input fast morph, stick-input morph; each plays once and ends naturally |
| GP | Normal Guard regression for all four branches; observe red-shield burst evidence |
| Consecutive use | At least two strict consecutive stick-input fast GPs with no degradation or alternation |
| Isolation | Morph animation and GP remain unchanged after a fast-morph chain |
| Follow-ups | X axe slam/roundslash and discharge; check super discharge where applicable |
| Lifetime | Action end, re-entry, rolling, sheathing, area transitions, and combat recovery |
| Loading | Clean automatic ExeFS loading without a debugger |
| JIT | Extended CPU-JIT-on play with no stutter, crash, or state contamination |

## 7. Promotion gate

If any item below is missing, the build should remain an experimental candidate or RC:

- no independent static/runtime proof of target-build addresses;
- stick-input fifth entry derived only from a cross-region address guess;
- machine-code readback passed but the move was never executed;
- GDB injection passed but clean ExeFS automatic loading did not;
- only one GP was tested without morph isolation, consecutive input, and recovery;
- no checksum, Title ID, or explicit save-state restriction.

## 8. Work outside the current porting scope

Giving GP to another weapon is not a regional port; it is new mechanism research. It requires further analysis of the relationship between `0x592` and the generic Guard/collision system, plus proof that the target weapon's lifecycle can safely establish and close the same defensive context. The current Charge Blade overlay must not be treated as a universal template.
