# Core's blood is red

## What

**On the Emitter Room (map 3) the port paints Core's blood in Core's own
reds, not in the colours of the original's tile.** Lysander drew the
blood with upper tile 123 of the RTP's `ChipSet1` (the game's ChipSet
`Future1`), a splash in salmon and pink that was there to borrow. It is
used twice: the cell (46,19), under the `DeadCore` event, and the
`CoreBlood` tile event at (64,45), from the earlier scene. In the port
both draw the same splash, pixel for pixel, in the four reds of the pool
under `DeadCore`'s pose (`Future3`, character 3): (100,20,10),
(150,50,20), (192,50,20) and (250,50,20), at 15 bits. The tile's nine
colours go to them by how light they are.

## Why

taro asked for it on 2026-11-16, after seeing the blood come out brown in
Mesen:

> Probably the best is a fix specific to this map. Since we can, and the
> original isn't particularly pretty, we could create a tile that is a
> copy-paste of the existing tile but with the colours of Core's blood.

The brown was the port's, not the game's: map 3 has more colours than its
background palettes hold, and Adeleine merged the splash's pinks into the
map's browns. Fixing the grouping in general is
[another question](../research/2026-11-16-the-palette-grouping.md); this
one is the targeted fix regardless. The original is not blood-coloured
either, so going back to its colours was not the aim.

On the console the cell comes out in a single red, (192,48,16): the map
still has room for one red among its background colours, not four. The
recipe keeps the four anyway, which is the drawing meant, and a better
grouping will show them without touching the patch. The rest of the map
does not get worse: with the splash's nine colours replaced by four, the
map has 71 distinct colours instead of 75 and Adeleine merges fewer, and
its background lands closer to what RM2k draws (measured by the task
below).

## How it applies

ChipSet 56, `ChipSet1_3`, in `abl/patches/manual/ChipSet/0056.json` in
the-advocate-ports: `Future1`'s record with its own id and name, and a
recipe with a single operation, `recolor`, which repaints upper tile 123
by colour. The operation is in `rm2k-snes/convert/src/chipsets.rs` of
the-advocate, with its test. Map 3 is forked to use it: the fork is the
dump's map with `chipset_id` 56 and the `id`, and its `.tiles` is the
dump's byte for byte.

The task is
[`abl-core-s-blood-turns-brown`](https://github.com/lautarodragan/the-advocate-work/blob/main/tasks/abl-core-s-blood-turns-brown.md).
If someone compares map 3 with the original and finds the blood a
different red: this is why.
