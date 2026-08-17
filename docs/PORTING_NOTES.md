# Porting Notes: MH4G JPN, MH4U USA, and MH4U EUR

## 1. Port the structure, not the addresses

The three builds share Action semantics and substantial function structure, so the verified behavioral design is portable:

```text
native GP mask timing
→ guard acceptance and GP recoil adjustment
→ red-shield condition after successful fast contact
→ charge-result classification and direct native burst submission
```

Function addresses, player-root slots (where the current player pointer is stored), branch distances, and usable blank code areas, or code caves, can differ. A port must not be made by changing only the Title ID, renaming an archive, assuming one global address delta, or copying a Gateshark address from another region.

The actual mappings demonstrate why one global delta is unsafe. From JPN to USA, for example:

```text
B56160 → B6D440   +0x172E0
B05488 → B1C6A0   +0x17218
CA92A8 → CC4350   +0x1B0A8
```

Those deltas differ. The selected USA-to-EUR core functions later happened to map at `+0x50`, but that was the result of independent signature verification for each function, not a presumed regional rule. Player-root slots, caves, and all other code still require separate checks.

## 2. Core mapping for the released builds

| Item | MH4G JPN/localized v1.2 | MH4U USA v1.1 | MH4U EUR v1.1 |
|---|---:|---:|---:|
| Title ID | `000400000011D700` | `0004000000126300` | `0004000000126100` |
| Player-root slot | `0106C3F4` | `081C7CD0` | `081C8140` |
| GP-mask helper hook | `00B56160` | `00B6D440` | `00B6D490` |
| Fast-contact hook | `00B05538` | `00B1C750` | `00B1C7A0` |
| Charge-result hook | `00B05488` | `00B1C6A0` | `00B1C6F0` |
| Native burst submitter | `00CA92A8` | `00CC4350` | `00CC43A0` |
| Standalone-fast cave | `00DCDEF8..00DCDFFF` | `00DECEF8..00DECFFF` | `00DECEF8..00DECFFF` |
| Combined cave | `00DCDCE8..00DCDFFF` | `00DECCE8..00DECFFF` | `00DECCE8..00DECFFF` |

These addresses belong only to the exact builds named in the table.

## 3. Porting workflow

1. **Export the target build's runtime image.** Verify the game and region first. If Azahar GDB cannot transfer a large block, use bounded chunked exports; do not interpret a failed `find` as proof that a target is absent.
2. **Locate functions by structure.** Use complete instruction sequences, neighboring control flow, call relationships, and observed Action behavior. A single constant or instruction is not a sufficient signature.
3. **Validate a cave.** It must be large enough, fully zero at runtime, free of known literals and branch targets, ARM-aligned, preflightable, and fully clearable during restoration.
4. **Recalculate every jump.** Re-encode each hook-to-cave jump, the return to the original function, the linked call to the native burst submitter, embedded absolute addresses such as the player-root slot, and every conditional or long-distance trampoline target.
5. **Run static validation before installation.** Confirm displaced-instruction replay, decoded branch targets, CPSR/register/LR preservation, literals, cave size and hash, IPS round trip, and deterministic rebuilds.
6. **Perform bounded runtime acceptance.** Use cold start, read-only preflight, full readback, minimal no-contact smoke, feature matrix, final status, and safe restoration.

## 4. Minimum runtime matrix

Standalone Fast Morph GP v4 must cover both input-branch animations, non-red GP without burst, red GP with one burst, AED/axe-slam follow-ups, morph/GP/held-R regression, final state integrity, and restoration.

The combined package must additionally cover short, normal, and automatic charge releases; non-red small, medium, and large recoil without burst; red small, medium, and large recoil with one burst; continued charging after small/medium recoil; interruption after large recoil; and confirmation that the charge-result filter does not affect held-R guarding or morph attacks.

Multi-hit attacks remain an explicit test boundary and must not be described as exhaustively covered.

## 5. EUR wrong-region cheat case

Two EUR climbing crashes at `00B96990` were eventually traced to an external cheat that used USA address `00B96958` on the EUR build. EUR normally contains required instruction `E58D1008` there, but the cheat replaced it with `E3A00001`. The USA target instruction `E3A01001` corresponds to EUR address `00B969A8`, not `00B96958`. Disabling a UI checkbox did not undo the existing runtime write; a full cold restart was required.

After the bad write was removed, the same GP candidate remained installed and the same route was climbed five times without a crash, followed by a passing feature matrix. The lesson is broader than this one code: record every simultaneous mutation, do not reuse regional addresses, and do not assume that disabling a code restores already-written machine code.

## 6. Release engineering

Every region and package type needs its own filename, Title ID, IPS hash, cave hash, archive hash, bilingual README, manifest, internal checksums, and explicit runtime-test status. A combined README must state that Fast Morph GP v4 is already included; a standalone README must state that charge GP is not included. RC, pending, or static-only artifacts must never be labeled as formal releases.
