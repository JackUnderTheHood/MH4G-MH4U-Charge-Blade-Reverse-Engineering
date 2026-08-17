# Release Notes

## Fast Morph GP v4

V4 is the final native-timing implementation of Fast Morph GP.

Unlike v3, it no longer uses five hooks and a resource overlay to substitute action identity. Both native fast Actions and animations are retained. The patch makes GP active during an existing native state update, and a successful-contact hook enters the native phial-burst path only when red shield is active.

Key changes:

- five hooks reduced to two;
- 640-byte overlay replaced with a 264-byte cave;
- no motion-identity mutation;
- explicit separation of non-red GP and red-shield burst behavior;
- no logger, counter, or per-frame persistent write;
- independent runtime acceptance on JPN, USA, and EUR.

V4 supersedes v3. Fully close Azahar before replacement and do not load save states across patch versions.

## Double Charge Slash GP v1

This is the combined package:

- Fast Morph GP v4 is already included;
- sword-mode charge hold gains a GP;
- small/medium recoil continues charging, while large recoil interrupts it;
- a successful red-shield GP creates one native burst; no red shield produces none.

The package uses a shared GP dispatcher and must not be installed together with standalone Fast Morph GP v4.

## Test boundary

Morph attacks, native GP, held-R guarding, primary follow-ups, charge releases, red-shield conditions, and all three recoil classes were covered. Multi-hit monster attacks did not receive dedicated coverage; no repeated burst was observed, but no exhaustive claim is made.

See [ARTIFACTS.md](ARTIFACTS.md) for final checksums.
