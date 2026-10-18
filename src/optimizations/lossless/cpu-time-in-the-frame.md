# Lossless: CPU time in the frame

## The hottest NPC code in bank 1, unrolled

What runs for every NPC row on every frame lives in bank 1, run from the
fast mirror, because bank 0 had no room to make it faster: hiding the 128
objects at the start of each frame is unrolled, and a row that moved is
moved to its place in the sorted table in one search and one block move
—`mvn`/`mvp`—, instead of re-sorting the table or swapping one place at a
time.

- **Saves**: scan lines of every frame, on every map.
- **Where**: `oam_hide` and `fit_far` in `runtime/far_npcs.s`.
- **More**: the header of `runtime/far_npcs.s`.

## The tint's tables are copied, not rebuilt

While a flash lasts, the palette is rebuilt every frame; the tint's three
channel tables —products and divisions— are kept in a pristine copy and
restored by copying it, and the flash's own table saves the tint's
computation per entry. A picture's tint reuses the same tables and
restores them the same way.

- **Saves**: CPU on every frame of a flash or a tinted picture.
- **Where**: `tint_tables`, `restore_tables` and `flash_channel_table` in
  `runtime/character_flash.s`; `pic_palette` in `runtime/picture.s`.
- **More**: the comments of those routines.

## A DSP write the register already has is skipped

The music decoder keeps a copy of what it wrote to each DSP register, and
does not write again a value the register already holds.

- **Saves**: writes through the SPC700's ports, every music frame.
- **Where**: `mus_write` and `MUS_SHADOW` in `runtime/music.s`.
- **More**: [Audio](../../internals/audio.md).

## The code runs from the FastROM mirror

Reset and the interrupts arrive in bank 0 and jump at once to its mirror
at `$80`, with `MEMSEL` set, where the cartridge is read at 3.58 MHz
instead of 2.68; every `jsl` into another code bank goes to its mirror
too. Everything fetched from the ROM, starting with the instructions, goes
a third faster.

- **Saves**: CPU on everything that runs.
- **Where**: `MEMSEL` and `FAST` in `runtime/registers.s`; the `jml` in
  `runtime/boot.s`; `MIRRORS` in `rm2k-snes/cartridge/src/banks.rs`.
- **More**: [ExHiROM, 8 MB](../../internals/exhirom.md), and the comment
  on `MEMSEL`.

## LZ2 decompressed with block moves

The decompressor does almost every LZ2 command with an `mvn`: a literal
from the stream, a repeat from the output onto the output, and a fill as
an `mvn` of the output onto itself shifted by one. Its body is written
once per output bank, because `mvn` carries its banks inside the
instruction, and the literal's `mvn`, whose source bank changes, is built
in RAM.

- **Saves**: the frames a map load and the return from a battle spend
  decompressing.
- **Where**: `lz2_grid` and `lz2_backdrop` in `runtime/decompress.s`;
  `LZ2_TRAMPOLINE_LIT` in `runtime/far_ram.s`.
- **More**: the header of `runtime/decompress.s`.

## One vblank budget, counted in poses

Everything uploaded to VRAM in the vblank takes from the same budget,
counted in poses: a tint or a flash, a line of the message window, a map
strip, a step of the ChipSet animation, a battle animation frame and a
painted map's slots subtract what they weigh, and the poses get what is
left. A pose that does not fit waits for the next vblank; the hero's goes
first.

- **Saves**: what would otherwise be lost —a DMA that lands outside the
  vblank writes nothing—, at the price of a character's pose arriving a
  frame later.
- **Where**: `flush_video` in `runtime/vblank.s`; `vq_cap` in
  `runtime/low_ram.s`; `anim_flush` in `runtime/chipset_animation.s`;
  `paint_flush` in `runtime/painted.s`; `vram_drain` in
  `runtime/load_graphics.s`.
- **More**: the comments in `flush_video`.

## The tint is a table per channel

The screen tint scales each channel the same way for every colour, so it
is computed for the 32 values a channel can take, one table per channel,
and each colour is looked up; the saturation, which needs the whole
colour, runs only when the tint asks for it.

- **Saves**: CPU —32 computations per channel instead of one per colour—.
- **Where**: `channel_table`, `build_palette`, `saturate_color` and
  `apply_tint` in `runtime/tint.s`.
- **More**: the comments of those routines.

## The screen flash walks its table

A flash moves every channel toward its colour by a fraction that only
depends on the distance, which goes down by one to the target and up by
one after it, so the products are walked along a row of a table instead
of computing an index for each entry.

- **Saves**: CPU on every frame of a flash.
- **Where**: `one_flash_table` in `runtime/screen_flash.s`.
- **More**: the comment on `one_flash_table`.

## A moving event's row and its slot are remembered

A move route slot remembers the row of its event in the NPC table, and
trusts it while it still belongs to that event; a parallel page's state
remembers its event's route slot; and the search for a slot stops at the
highest one used since the map loaded.

- **Saves**: CPU —searches that ran for every moving event and every
  parallel page, on every frame—.
- **Where**: `slot_npc` in `runtime/far_npcs.s`; `PAIR_SLOT` in
  `runtime/far_ram.s`; `route_ev_cap` in `runtime/low_ram.s`.
- **More**: the comments there.

## The events' passability is looked at only when some row imposes it

When the map loads, one pass says whether any event row carries
passability bits of its own —an event drawn with a tile—, and if none
does, the collision does not walk the rows to ask.

- **Saves**: CPU on every step that asks for passability.
- **Where**: `pass_look` and `pass_has` in `runtime/far_npcs.s`.
- **More**: the comment on `pass_look`.

## The NPC table is copied to RAM when the map loads

Of the map's tables, the NPC one is the only one copied to bank 0's RAM
instead of read through a pointer: `build_oam` reads it with an index at
several offsets, and that is one instruction per field instead of three.

- **Saves**: CPU on every frame.
- **Costs**: one DMA per map load, and bank 0's RAM.
- **Where**: `npc_buf` in `runtime/low_ram.s`; `load_npcs` in
  `runtime/load_graphics.s`.
- **More**: the comment on `npc_buf`.

## A picture's palette, only when its tone really changed

A picture whose tone moves over some frames rebuilds its palette only on
the frames the step changes the tone, not on every frame of the ramp.

- **Saves**: CPU —products and divisions for the same colours—.
- **Where**: `pic_ramp_step` in `runtime/picture_far.s`.
- **More**: the comment on `pic_ramp_step`.

## The timer counts seconds

The timer keeps seconds and the frames left in the second, not frames,
and its four digits are worked out when the second changes, so nothing
divides by 60 every frame.

- **Saves**: CPU on every frame the timer runs.
- **Where**: `timer_tick` and `timer_digits` in `runtime/timer.s`;
  `timer_sec` and `timer_sub` in `runtime/low_ram.s`.
- **More**: the comment on `timer_sec`.

## A fade rewrites only the volumes that changed

On each frame of a `FadeOutBGM` the music's voice volumes are scaled
again, but each is compared with what the previous frame's level gave and
written only if it differs: a long fade moves each volume once every so
many frames.

- **Saves**: writes through the SPC700's ports, each with its wait, on
  every frame of a fade.
- **Where**: `voices_refresh` and `volume_refresh` in `runtime/music.s`.
- **More**: the comment on `voices_refresh`.

## In mode 7, the music multiplies by a table of squares

The music scales its volumes with the PPU's multiplier, which in mode 7 is
the matrix being drawn; while a big picture zooms it multiplies by hand
instead, as two lookups in a table of quarter squares,
`m × f = q(m + f) − q(|f − m|)`, because a loop of shifts and adds took
longer than the music has before the vblank.

- **Saves**: CPU in `irq_music` during a zoom.
- **Costs**: ROM for the table, in a far bank.
- **Where**: `mpym` and `quarter_squares` in `runtime/music.s`.
- **More**: the comment on `mpym`.
