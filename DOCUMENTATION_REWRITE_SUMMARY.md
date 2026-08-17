# Documentation Rewrite Summary

Date: 2026-08-16

This documentation set was rebuilt around the final Fast Morph GP v4 and Sword-Mode Charge Slash GP v1 project state. The goal is to preserve the research history while presenting verified conclusions, release information, and remaining limits in a form that readers can follow without reconstructing the project from raw experiment logs.

## What was reorganized

- Rewrote the English and Chinese READMEs around package choice, installation, validated behavior, and safety notes.
- Moved the final mechanism explanation into Technical Analysis.
- Separated v3, the withdrawn “v4 documentation repack,” and the final two-hook v4 in Research History.
- Documented independent JPN, USA, and EUR address mapping and acceptance requirements in Porting Notes.
- Listed all six formal ZIP files, `code.ips` files, code-cave sizes, and SHA-256 identities separately.
- Stated clearly that standalone Fast Morph GP v4 and the combined Charge Slash GP v1 package are alternatives; the combined package already contains v4.
- Described input branches as fast morph without moving the left stick and fast morph while moving it, avoiding ambiguous neutral/directional edition names.
- Documented the Azahar 2126.0 research baseline, unstable conventional breakpoints/watchpoints, and the use of scripts, temporary hooks, and bounded loggers for primary collection.
- Recorded the contribution boundary between AI-assisted code work and human runtime operation and acceptance.
- Preserved Hazerou's contribution as the source of important early public MH4U Gateshark leads.

## Intentionally unchanged

### `current_state.md`

The original file is a live research archive containing failed candidates, provisional hypotheses, superseded interpretations, and operational records. Rewriting it would damage the evidence history, so an exact copy is preserved at:

```text
archive/current_state_original_2026-08-16.md
```

- Size: 1,005,740 bytes
- SHA-256: `6FFC5954A0460DDB0CF8F2F2416E403C1ACE15842B3D04C6A840FD7687539089`

### Raw local reports and experiment artifacts

Raw reports, logger output, release candidates, and failed experiments remain part of the local evidence set. They were not rewritten and were not all copied into this public documentation package. Public reading should begin with README, Technical Analysis, Research History, and Porting Notes; use the archive's references when a finer experiment trail is needed.

### Six formal release ZIP files

The accepted patch archives were not repacked during this documentation rewrite. Their published identities and checksums therefore remain unchanged. Any future edit to a README inside those ZIP files must be released as a new build with newly calculated hashes.

## Recommended reading order

1. `README.md`
2. `RELEASE_NOTES.md`
3. `docs/TECHNICAL_ANALYSIS.md`
4. `docs/RESEARCH_HISTORY.md`
5. `docs/PORTING_NOTES.md`
6. `ARTIFACTS.md`

The archived `current_state.md` is a Research Archive and should not be used as the public README.
