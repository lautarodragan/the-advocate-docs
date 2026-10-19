# Lossless: the map

## The 16x16 block grid

The map is not stored as a tilemap but as a grid of RM2k's 16x16 cells,
each pointing to a **block** that carries its four background cells, its
four cells of the layer above and its passability. Blocks repeat a lot,
and identical ones are stored once. Expanded, a big map's tilemap would
not fit in the cartridge at all.

- **Saves**: ROM —one grid entry per 16x16 cell instead of four tilemap
  entries per layer—; and since passability travels in the block, **the
  whole collision table** —one byte per cell— is gone.
- **Where**: `materials.rs` in `rm2k-snes/mapdata/`; the readers, `ptr_more`
  and `ptr_minus` in the runtime.
- **More**: [Maps](../../internals/maps.md).

## A pointer that walks with the hero

Where the hero is in the grid is a 24-bit pointer that moves one cell or
one row per step, not an index: nothing multiplies or builds an address
from scratch, even though the grid crosses banks.

- **Saves**: CPU on every step and every passability question.
- **Where**: `ptr_more` and `ptr_minus`, the only routines that know how
  the grid crosses a bank.
- **More**: [Maps](../../internals/maps.md).

## The grid, its blocks and the bytecode, in LZ2 and decompressed into WRAM

The grid travels in the cartridge as LZ2 and is decompressed whole into
the upper halves of the two WRAM banks when the map loads; the block
table, its passability and the event bytecode go right behind it, in the
same load. The upper half of two consecutive WRAM banks has the shape of
two LoROM banks, so the readers did not change.

- **Saves**: ROM, most of the grids' weight, keeping direct access cell
  by cell.
- **Costs**: 32 KB of far RAM in `$7E`, and the decompression on every map
  load and on returning from a battle.
- **Where**: `GRID` in `runtime/far_ram.s`, `lz2_grid` in
  `runtime/decompress.s`, `map_ram_pointers` in `record.rs` of
  `rm2k-snes/mapdata/`.
- **More**: [The grid lives in WRAM](the-map/the-grid-in-wram.md), with what it
  costs, how to know it needs reverting and how;
  [Maps](../../internals/maps.md).

## The map's tiles in LZ2

The tiles of the background, of the layer above, of the sprite bank and
the drawings of the ChipSet animation travel as LZ2, are decompressed into
a WRAM buffer and uploaded with the same DMA as before. A background too
big for the buffer goes in two streams.

- **Saves**: ROM.
- **Where**: `materials.rs` in `rm2k-snes/mapdata/`; the format and the
  compressor, `snes/lz2/`; `lz2_backdrop` in `runtime/decompress.s`.
- **More**: [Maps](../../internals/maps.md), [The VRAM split](../../internals/vram.md).

## Only the tiles the map draws go to VRAM

The ROM does not take the whole ChipSet to VRAM, as RM2k does, but the
unique 8x8 tiles of what the map draws: the converter renders the map and
takes the tiles from there. An expanded ChipSet is hundreds of 8x8 tiles,
and they do not fit in what is left for the background next to the
message window. Nothing changes on screen: each map draws as in RM2k.

On 2026-09-07, counting only the ids some map of ABL uses, `ChipSet3`
gave 420 tiles and `BLchp1` 523; map 1 drew 341 and map 2, 278, against
the 420 of the ChipSet they share, and the pool of the two, 437, fit in no
VRAM layout of that day. The full count is in [the study on RM2k graphics
on the SNES](../../research/graphics-and-sound-from-rm2k-to-the-snes.md),
"Sharing tiles between maps".

- **Saves**: VRAM.
- **Costs**: two maps with the same ChipSet do not share their tiles,
  which would save cartridge bytes and the VRAM reload between them.
- **When it stops being enough**: a game whose maps use more distinct
  background tiles than any layout holds. No layout saves that: VRAM would
  have to be reloaded while the map scrolls, a different engine from this
  one —except for a painted map, below—.
- **Where**: Wally, `rm2k-snes/wally/`; the layout, `vram_layout` in
  `vram.rs` of `rm2k-snes/mapdata/`.
- **More**: [The VRAM split](../../internals/vram.md).

## The VRAM layout is chosen map by map

There is no single split of the 64 KB that serves every map, so the
converter picks, for each map, the first of a fixed list of layouts where
its background, its sprites and its layer above fit, and emits the
register values ready-made. New layouts go at the end, so no map that
already fit changes.

- **Saves**: VRAM —each map gets the split it needs, within the
  hardware's alignments—.
- **Where**: `vram_layout` and `LAYOUT_WINDOW*` in `vram.rs` of
  `rm2k-snes/mapdata/`.
- **More**: [The VRAM split](../../internals/vram.md).

## The CGRAM split is chosen map by map, background first

The lower half of CGRAM is shared by the 8-bit background and the 4-bit
layer above. The converter decides the cut for each map: the background
keeps the palettes it needs, and the layer above takes what is left.

- **Saves**: CGRAM —no fixed cut serves every map—; merging colours only
  where nothing else is left.
- **Where**: `convert_with` in `rm2k-snes/wally/src/background/layers.rs`.
- **More**: [The CGRAM split](../../internals/cgram.md).

## Opaque fixed events baked into the map

An event drawn with a ChipSet tile that is also opaque and never moves is
painted into the map, and its row is hidden from the sprite pass.

- **Saves**: sprite tiles, OAM entries and CPU in `build_oam`.
- **Where**: `baking::bakeable` in Wally (`rm2k-snes/wally/src/baking.rs`);
  `build_oam` skips the hidden row.
- **More**: [Events](../../internals/events.md).

## A painted map keeps only what the screen shows

A map whose panorama *is* the map goes in BG1 in mode 1, one tile per 8x8
cell, and VRAM holds only the cells the screen can show, in a ring of
slots that the camera fills as it moves. A spare row uploaded ahead keeps
the vblank short.

- **Saves**: VRAM —a painted map has far more distinct cells than any
  layout holds—.
- **Where**: `runtime/painted.s`, `paint_table` in `panorama.rs` of
  `rm2k-snes/mapdata/`. Only a cartridge with a painted map assembles it.
- **More**: [The VRAM split](../../internals/vram.md#a-painted-map-keeps-only-what-the-screen-shows).

## A panorama gets only the layer it needs

For each map the tools choose how to show its panorama: not at all if it
is never seen, baked into the background if it does not move against the
map, in the second layer if the map draws little above the hero, or with
the map in mode 1 and the panorama on a third layer.

- **Saves**: VRAM and a background layer, in the maps that do not need
  one.
- **Where**: `Way` in `rm2k-snes/wally/src/background/panorama.rs`;
  `runtime/pano.s`.
- **More**: [Mode 3 by default](../../internals/mode-3.md),
  [Panoramas](../../research/2026-10-05-panoramas.md).

## The pictures that stay put are drawn into the map

With `[graphics] map_pictures = true`, the `ShowPicture` pictures that
stay where they are on a map —the lights, shadows, mists and what goes in
front of the hero in a game drawn as a painting— are drawn into the map
when the cartridge is built: blended into its panorama or its tiles, and
what the opaque ones cover put in front of the hero by the priority bit.
Their commands are then accepted without being done.

- **Saves**: the layer, the sprites and the palettes the pictures would
  take at run time —in mode 3 there is no layer left for them, and colour
  math does not reach the characters—, and the CPU of drawing them.
- **Not lossless when**: the picture moves, fades, is erased or shades the
  characters, what goes in front of the hero falls on 8x8 cells, and the
  colours and tiles it blends in count against the map's, which may merge
  or fuse some.
- **Where**: `map_pictures.rs` in `rm2k-snes/convert/src/` and in
  `rm2k-snes/wally/src/`.
- **More**: [Pictures drawn into the map](../../divergences/pictures-drawn-into-the-map.md).

## The tilemap is a window the camera fills at its edges

The map's tilemaps in VRAM are a window of 64 columns by 32 rows that
wraps around, not the map: each tick builds, outside the vblank, only the
row and the column the camera just uncovered, and the next vblank uploads
each in one DMA, because the strips are laid out in the order VRAM wants
them.

- **Saves**: VRAM —a map of any size takes one window— and vblank time,
  since the strips are built while the screen is drawn.
- **Costs**: what changes under a still camera is not redrawn by itself:
  a `TileSubstitution` redraws the window one column per frame, on the
  frames the camera leaves free.
- **Where**: `stream_map`, `build_column` and `build_row` in
  `runtime/map_window.s`; `push_strips` in `runtime/vblank.s`;
  `sub_refresh` in `runtime/low_ram.s`.
- **More**: the comments of those routines.

## The ChipSet animation is one DMA, on the tick it changes

The tiles that the water and the animated tiles change go in a contiguous
run at the end of the map's background bank, shared with no still cell,
so a step of the animation overwrites those tiles with a single DMA and
leaves the tilemap alone. Whether a tick is a step comes from the same
division that says where in the cycle it is.

- **Saves**: vblank time —no tilemap rewrite, and no DMA on the ticks
  that change nothing—. What it saves in RAM is lossy: see [the ChipSet
  animation's clocks come from the game
  tick](../lossy/tiles.md#the-chipset-animations-clocks-come-from-the-game-tick).
- **Where**: `anim_tick`, `anim_flush` and `anim_upload` in
  `runtime/chipset_animation.s`; the run, `layers.rs` in
  `rm2k-snes/wally/src/background/`.
- **More**: the header of `runtime/chipset_animation.s`.

## A cell's terrain is its block's, and a flat map's is one byte

`StoreTerrainID` reads a cell's terrain from its lower layer's tile, so the
converter puts the terrain in the block's key and carries one byte per
block, not per cell; and a map whose blocks all have the same terrain
carries no table at all, only that terrain, a block shared by every such
map. A `TileSubstitution` changes the terrain the same way it changes the
block.

- **Saves**: ROM and the WRAM behind the grid —a table per block instead
  of per cell, and none on a flat map—.
- **Where**: `terrain_plane` and `map_terrain` in
  `rm2k-snes/mapdata/src/record.rs`; `op_terrain` in
  `runtime/interpreter_map.s`, `M_TERRAIN_PLANE`.
- **More**: the comment on `op_terrain`.

## The panorama's tilemap is a ring the camera fills too

A panorama on a layer of its own does not fit in the 64x32 tilemap
—RM2k's are 640 pixels wide and more—, so its tilemap is a ring like the
map's: a running position that wraps where both the tilemap's width and
the panorama's do, and each frame at most the column and the row that
came into view, built outside the vblank and uploaded in it.

- **Saves**: VRAM —one tilemap for a panorama of any size— and vblank
  time.
- **Where**: `pano_tick`, `pano_vblank` and `PANO_UX` in
  `runtime/pano.s`.
- **More**: the header of `runtime/pano.s`.

## A grid of at most 256 blocks takes one byte a cell

A cell of the grid is the number of its block, and a map with at most 256
distinct blocks stores it in one byte instead of two. Every map that can
goes narrow, not only those that need it, so every map plays that path.

- **Saves**: WRAM —half the grid on most maps— and with it the maps that
  fit at all: the 64 KB hold twice the cells.
- **Where**: `cell_bytes` in `materials.rs` of `rm2k-snes/mapdata/`;
  `M_GRID_NARROW`, which `enter_map_no_events` in `runtime/enter_map.s`
  turns into `cell_bytes` and `cell_mask`, read by `cell_ptr` in
  `runtime/hero.s` and by `build_column`, `build_row` and
  `cells_to_bytes` in `runtime/map_window.s`.
- **More**: [Maps](../../internals/maps.md#the-grid-lives-in-wram-and-the-cartridge-carries-it-compressed).

## A cell's passability is worked out when the cartridge is built

RM2k asks, on every step, for the passability of the cell's lower and
upper tiles and which of the two rules; the converter answers that for
each cell, with both layers resolved as the hero meets them, and the
block carries the answer as four bits, one per side. A step reads one
byte.

- **Saves**: CPU on every step and every passability question, and the
  ChipSet's passability tables in the cartridge.
- **Where**: `blocks_of` in `rm2k-snes/wally/src/layers.rs`, with
  `rm2k_rules::passability::cell`; `pass_of_ptr`, `can_leave` and
  `can_enter` in `runtime/hero.s`.
- **More**: the header of `runtime/hero.s`.

## The load buffer is lent while the map is played

The 24 KB where a map load decompresses each tile block before its DMA
are idle once the map is on screen, so they hold what lives only then:
the battle animation's frames as they are decompressed, and the bytecode
of a page that goes on after a teleport. The ChipSet animation's drawings
live right above it; a battle takes the whole bank, and coming back
decompresses them again.

- **Saves**: far RAM —none of them has a buffer of its own—.
- **Costs**: what borrows it is rebuilt after a load, a screen or a
  battle: `back_to_map` decompresses the carried bytecode again.
- **Where**: `LZ2_BUFFER`, `CODE_CARRY` and `ANIM_DRAWINGS` in
  `runtime/far_ram.s`, with the asserts at the end of
  `runtime/decompress.s`; `code_carry_after` there.
- **More**: [The WRAM and SRAM map, measured](../../research/2026-10-15-the-wram-and-sram-map.md#what-is-shared-in-time-and-what-is-idle-when).

## Coming back from a screen reloads only what it overwrote

The menu, the shop and the slot screen draw only in tiles and a tilemap
that fall inside the map's background tile bank in every VRAM layout, so
closing one uploads that bank and its palettes again, not the whole map:
the tilemap window, the sprites and the message window stay. A battle
overwrites far more and comes back by its own, longer way.

- **Saves**: the frames of a full reload on every screen that closes —the
  64 columns of `fill_window` among them—.
- **Where**: `back_to_map` and `back_from_battle` in
  `runtime/enter_map.s`; `SLOT_VRAM` and `SLOT_MAP`, which the converter
  checks against every layout in `vram.rs` of `rm2k-snes/mapdata/`.
- **More**: the comments of those two routines.

## The window and the tint are filled in the fade's black ticks

Inside a teleport's fade, the load leaves for the black ticks after it
what only the screen needs: clearing the tilemaps, the window's 64
columns, the panorama's ring and the painted slots, and the tint. Each
black tick does what fits before the vblank, with the screen in forced
blanking, which with the brightness at zero is the same black; the tint
goes to CGRAM in the next vblank, as an event's does. Whatever is left is
done on the last black tick, so the ramp up shows the whole map as before.

- **Saves**: four to eight frames of every load inside a fade, when the
  package does not need the black ticks.
- **Where**: `tail_pump` and `LOAD_DEFER` in `runtime/enter_map.s`;
  `window_start` and `fill_columns` in `runtime/map_window.s`.
- **More**: the comment on `tail_pump`.
