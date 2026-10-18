# Lossless: battle

## The scene and the windows are one layer

In battle the scene and the windows share BG1 in mode 3, the windows in
the rows below the scene, so only BG1's horizontal scroll goes by HDMA and
no mode or base changes mid-frame.

- **Saves**: VRAM and HDMA channels.
- **Where**: `battle.rs` in `rm2k-snes/screens/`; `ba_win_tick` in
  `runtime/battle_scene.s`.
- **More**: [The VRAM split](../../internals/vram.md#battle-doesnt-use-the-layout).

## Each distinct troop page once

A game copies its battle systems' pages into every troop, so the troops'
pages are compiled and deduplicated by their bytecode: each distinct body
is stored once, and every troop that has it points to it.

- **Saves**: ROM in the troops' bytecode.
- **Where**: `rm2k-snes/events/src/troops.rs`.
- **More**: [What troop pages do not model](../../divergences/what-troop-pages-do-not-model.md).

## The battlers and the light backdrops in LZ2

The troop's battlers and the backdrops the ROM carries travel as LZ2 and
are decompressed when the battle starts.

- **Saves**: ROM.
- **Where**: `rm2k-snes/convert/src/scene.rs`; `runtime/battle_scene.s`.
- **More**: [The `.msu` file](../../internals/the-msu-file.md#what-does-not-fit-in-the-rom).

## Encounters and experience, precomputed

The encounter odds are integer arithmetic the converter prepares: the
reciprocal of each map's `encounter_steps`, in 1/65536, and the step at
which each row of the table is crossed; the ROM does not divide. And a
victory's experience comes summed per troop, since every enemy in a
victory is dead.

- **Saves**: CPU —no division on the console—.
- **Where**: `base` and `step_thresholds` in
  `rm2k-snes/battle/src/encounters.rs`, read by `runtime/encounters.s`;
  `troop_exp` in `rm2k-snes/battle/src/rules.rs`, read by `ba_exp_allocate`
  in `runtime/battle_loop.s`.
- **More**: the comments of those routines.

## Experience only for the actors who can fight, each curve once

Only the actors who can be in the party get an experience slot —four bytes
of experience and level, and what the materials added—; any other keeps
its database level forever and answers from the cartridge's tables. The
experience each level asks for is stored once per distinct curve, not per
actor.

- **Saves**: far RAM and SRAM —the slots travel in the save— and ROM.
- **Where**: `tables` and `slots` in
  `rm2k-snes/battle/src/experience.rs`, `actor_exp_k` and `exp_table`;
  `actor_level_of`, `level_param` and `exp_add` in `runtime/level.s`.
- **More**: the comment on `FAR_EXP` in `runtime/far_ram.s`.

## The battlers in the grid's half of `$7E`

The troop's battlers are decompressed into the half of `$7E` the grid
uses, since `back_from_battle` decompresses the grid again anyway.

- **Saves**: far RAM.
- **Where**: `SCN_BATTLERS` in `runtime/battle_scene.s`.
- **More**: [The grid lives in WRAM](the-map/the-grid-in-wram.md#what-is-lost).

## A dead enemy redraws only its cells

The battle scene is composed on the console, and a clean copy of the
backdrop stays in far RAM beside it. When an enemy dies, only the cells
it covered are composed again —the clean backdrop, and on top the enemies
still alive, clipped to those cells and in drawing order— and only those
rows go up, up to 2 KB per vblank, one enemy at a time.

- **Saves**: CPU and vblank time —recomposing and uploading the whole
  scene for each enemy that dies—.
- **Where**: `scene_tick` and `SCN_SUB_BUDGET` in
  `runtime/battle_scene.s`; `SCN_CLEAN` and `SCN_SCENE` there too.
- **More**: the comment on `scene_tick`. That the enemy goes all at once
  instead of fading as in RM2k is not this entry's: it is a divergence
  the book does not tell yet.

## The scene is composed on the console, from each backdrop and battler once

The cartridge does not carry a picture of each troop's scene: it carries
each backdrop once and each battler once, and the console composes the
scene when the battle starts, in far RAM. Two enemies with the same
battler share its palette and its decompressed bytes.

- **Saves**: ROM —a scene per troop—, and CGRAM and decompression time
  for repeated enemies.
- **Where**: `scene_upload` and `scn_battlers_load` in
  `runtime/battle_scene.s`; `rm2k-snes/convert/src/scene.rs`.
- **More**: the header of `scene.rs`.

## A battler is its mask and four planes

The scene is 8 bits a pixel, but a battler stores only its mask and the
four planes of its drawing: its colours are one palette of sixteen, so
the other four planes are the palette's number, the same on every pixel
it draws, and the console sets them while placing it.

- **Saves**: ROM —close to half of each battler—.
- **Where**: `BATTLER_PLANES` in `rm2k-snes/convert/src/scene.rs`;
  `ED_P45`, `ED_P67` and `SCN_LOOP` in `runtime/battle_scene.s`.
- **More**: the comments of `scene_set`.

## The battlers are placed with one loop per shift

A battler that does not start on a tile edge is shifted by a fixed number
of bits, so the placing loop is written eight times, once per shift, with
its shifts unrolled; a column with nothing of the battler is skipped; and
the loop runs on a borrowed direct page aligned to 256, which saves a
cycle on each of its accesses.

- **Saves**: CPU —the frames composing the scene takes, when the battle
  starts and each time an enemy is erased or appears—.
- **Costs**: ROM for the eight copies of the loop.
- **Where**: `SCN_LOOP`, `scn_loops` and `scn_dp_take` in
  `runtime/battle_scene.s`.
- **More**: the comments above `SCN_LOOP` and on the borrowed direct page.

## The light backdrop is doubled through a table

A backdrop the ROM carries goes at half size and 4 bits: its four upper
planes, the same on every pixel, go into every tile by DMA, and its lower
ones are doubled by the CPU through a table of 256 words that turns each
byte into the two it becomes. The converter leaves the bytes in the order
they are read, so the loop only moves forward.

- **Saves**: CPU when a battle starts without the `.msu`.
- **Where**: `scn_fill` and `scn_double` in `runtime/battle_scene.s`;
  `fallback_bytes` in `rm2k-snes/convert/src/scene.rs`.
- **More**: what that backdrop loses is in [Lossy](../lossy/pictures-and-animations.md#light-battle-backdrops-in-the-rom).

## The windows: a band per layout, and each bar only where it changes

The battle's windows are stored as the tilemap rows of each of their
layouts, and each cursor bar as only the cells where it differs from its
layout, a piece equal to another stored once. They go up only when the
layout or a bar changes, and then only the band and the bars' cells.

- **Saves**: ROM, and vblank time.
- **Where**: `battle_windows` in `rm2k-snes/screens/src/battle.rs`;
  `ba_win_tick` and `ba_win_upload` in `runtime/battle_scene.s`.
- **More**: the comment on `battle_windows`.

## The critical hit is looked up

RM2k adds an actor's own critical chance and its weapon's in floating
point, so the console does not add them: it reads the result from a table
with a row per distinct own chance and a column per distinct weapon
bonus. The columns go by bonus, so the better of two weapons is the
higher column, and each actor's is worked out once, when the battle
starts.

- **Saves**: CPU and code —no floating point on the console—, and ROM,
  since the table is only as big as the distinct values.
- **Where**: `critical_percent` in `rm2k-snes/battle/src/rules.rs`;
  `crit_table` in `rm2k-snes/mapdata/src/emit.rs`; `ba_crit_of` in
  `runtime/battle.s`.
- **More**: the comment on `crit_table` in `emit.rs`.

## The battle's tint and flash, composed once per change

A flash or a tint in battle runs the scene's colours through the tint's
three channel tables composed with the flash's, so each colour is three
lookups; the palette is built only on a frame where the tint or the flash
changed, and only over the colours the backdrop draws —sixteen for the
light one—.

- **Saves**: CPU on every frame of a flash in battle.
- **Where**: `ba_tone_tick` and `ba_tone_build` in `runtime/battle_tone.s`.
- **More**: [The tint is a table per channel](cpu-time-in-the-frame.md#the-tint-is-a-table-per-channel),
  and the header of `runtime/battle_tone.s`.
