# The WRAM map

The console has 128 KB of WRAM, two banks, `$7E` and `$7F`, and the first
8 KB of `$7E` are seen again at `$0000`–`$1FFF` of every bank that has
ROM: that is **bank 0's RAM**, the only RAM that absolute addressing
reaches with the data bank at zero. Everything else is **far RAM**,
reached with 24-bit addresses (`lda f:`), through the `WMADD`/`WMDATA`
window, or with the data bank set to `$7E` or `$7F` —in bank `$7E`,
`$0000`–`$1FFF` *is* bank 0's RAM, so code that runs there sees both; in
`$7F` it is not—.

The map, from the bottom up, with the area of the `.cfg` each part is in
and the segment the runtime declares it with:

```text
         area     segment
$7E0000  ZP       ZEROPAGE   the direct page
$7E0100  RAM      BSS        bank 0's RAM
         STACK               the stack, at the top of bank 0's RAM
$7E2000  FARRAM   FARBSS     far RAM: every subsystem's state
                  SWITCHES   the switch table, at FARBSS's end
$7E8000  GRIDLO   GRID       the map's grid, its first 32 KB
$7F0000  LOADBUF  LOADBUF    LZ2_BUFFER, where a map's streams are
                             decompressed, and ANIM_DRAWINGS, the
                             ChipSet animation's drawings
$7F8000  GRIDHI   GRIDHI     the grid past 32 KB, then what lives behind
                             it, laid out by the converter
                  GRIDTOP    GRID_TOP, at the end of the half
```

**All of it is placed by the linker**: no address of the WRAM is written
in the runtime. The areas are `WRAM_AREAS` and the stack's in `banks.rs`
of `rm2k-snes/cartridge/`, which writes the `.cfg`; each segment's `.res`
is in the file of the subsystem that owns it. What has a reason to be at
a given place says it there: the grid's halves are where the converter
writes them (see [The upper halves](#the-upper-halves-the-grid-and-the-buffers)),
and the stack is at the top of bank 0's RAM. The one long number left in
the runtime is far RAM's bank, `$7E`, which is not a place: it is where
bank 0's RAM shows again, and what the code that runs with the data bank
there counts on.

What each linker segment has left, per cartridge, is said by `just space`
in each game's directory of the ports: this page does not say it, because
it changes with every change.

## Far RAM and bank 0's

**In `ZEROPAGE`, `BSS`, `FARBSS` and `SWITCHES` a variable is a `.res`** in
the file of the subsystem that owns it, and the link fails when one of
them is full instead of overwriting a neighbour. Why, and the
traps of the `far` segment (`.loword` with the data bank at `$7E`, `f:`
for a label used before it is declared): [RAM is placed by the
linker](../decisions/ram-is-placed-by-the-linker.md). What the game sizes
—the variables, the items, the actors' tables— is declared in
`far_ram_game.s`, after the cartridge's `.inc` says how big it is; the
rest, in each subsystem's file, at its own size.

* **The zero page** is for what is read through a pointer —`[ptr], y`,
  which only exists there—, and for the hottest variables, one byte and
  one cycle cheaper than absolute.
* **The `BSS`** holds what is sent by DMA from bank 0 —the OAM, the
  palettes, the strips of the tilemap— and the tables the frame walks:
  the events' positions, their flags, the NPC table copied from the
  cartridge. Its tables are sized by the runtime's caps (`EV_MAX`,
  `NPC_MAX`…), not by the game, so it is the same in every game. What a
  map that has more events than those rows loses, and before which other
  step: [the converter's stages](the-converters-stages.md#1-the-rams-trim).
* **`FARBSS`** is the whole of `$7E2000`–`$7E7FFF` but the switch table:
  the interpreter's parallel pages, the variables and the inventory, the
  battle's state, the text layer of the screens, the audio's allocation,
  and everything else that lives longer than a load. Many of its regions
  have uppercase sixteen-bit names (`FAR_BATTLE` is `.loword(far_battle)`)
  because their code runs with the data bank at `$7E`.
* **What travels in the save** is listed by `save_table`, whatever
  segment it is in: see `bank1.s`.

## The upper halves: the grid and the buffers

The upper half of `$7E` and the upper half of `$7F` are the `GRIDLO` and
`GRIDHI` areas, and they have the shape of two consecutive LoROM banks
—`$8000`–`$FFFF` of two banks in a row—, which is what lets `ptr_more` and
`ptr_minus` walk a grid of more than 32 KB: see [The grid lives in
WRAM](../optimizations/lossless/the-map/the-grid-in-wram.md).

**They are where they are because the converter writes them as numbers**,
into what it compresses before anything is linked —the drawn-in events'
cells, the page tables' code pointers—. So the two addresses live twice,
on purpose: in `banks.rs`, which gives them to the linker, and in
`materials.rs` of `rm2k-snes/mapdata/`, as `GRID_AT` and `GRID_HIGH_AT`,
which the converter writes in the cartridge's `.inc`; `far_ram_game.s`
asserts that they are the same, and the link fails if not.

**`$7E8000`–`$7EFFFF` is the grid**, the `GRID` segment, one or two bytes
a cell —the record's `M_GRID_NARROW`—, decompressed by `lz2_grid` on every
map load. A painted map, whose grid is small, keeps its queue of DMAs at
the end of that bank: `PAINT_Q` in `painted.s` is `GRID + PAINT_ROOM`,
the converter's number for how much such a grid may take, which it
refuses a map past.

**`$7F0000`–`$7F7FFF` is a work area**, the `LOADBUF` area between the
grid's halves, free once a map is loaded; its segment holds two `.res`,
`LZ2_BUFFER` and `ANIM_DRAWINGS`, and what borrows them is an offset of
them with an `.assert` of how far it goes:

* `LZ2_BUFFER`, where the background's tiles, the layer above's, the
  sprites' and the panorama's are decompressed before going up to VRAM by
  DMA —the panorama's also when a `ChangePBG` swaps it with the map up,
  and then `pano_image` rebuilds the buffer's tenants—; and once the
  map is loaded, in that same place, the bytecode a page carries across
  a teleport (`CODE_CARRY`, see [Events](events.md)), the battle
  animation's frames, a big picture's tilemap, the wide text stage, the
  menu's rows, the debug screen's caches while it is open, and the image
  of a full screen (`SCREEN_BUFFER`) —each one
  `.assert`s it does not reach the next—.
* `ANIM_DRAWINGS`, at `$7F6000`, the ChipSet animation's drawings, which
  stay for as long as the map lasts because they go up every time the
  clock changes; a big picture's palette and waves live there while it
  is shown.

**`$7F8000`–`$7FFFFF` is the grid past its first 32 KB, and behind it
what the CPU reads of the map while it is played**, in this order, each
right where the one before ended —the `GRIDHI` segment, whose `.res` is
the whole half but `GRID_TOP`—:

```text
GRID_HIGH   the grid past 32 KB, if it goes that far
            the block table and each block's passability
            the event bytecode                  (map_code)
            the event page table                (map_pages)
            ...free...
GRID_TOP    GOLD_BUF                            the inn's gold window's rows
            POSED_REG, POSED_T                  the drawn-in events' runs
$7FFFFF
```

`lz2_grid` decompresses them in a row, one stream after another, and does
not compute any address: **the converter lays them out** and writes where
each one lands in the map's record and in `map_code` (`map_ram_pointers`
and `pages_in_wram` in `record.rs` of `rm2k-snes/mapdata/`).
`test_decompressed_load.py`, in ABL, checks on every map that what is
there is what the converter says. It stops the build if a map's grid and
what goes behind it do not fit below `GRID_TOP` (`check_map` in
`materials.rs`); the bytecode and the page table are the exceptions, which
stay in the cartridge when they do not fit —see [Events](events.md)—.

### `GRID_TOP`: a segment with named tenants

The last bytes of the half are **`GRID_TOP`**, the `GRIDTOP` segment, after
`GRIDHI` in the same area, which no map's grid and nothing behind it
reach, on any map: the converter keeps them out (`BEHIND_ROOM` in
`materials.rs`). Each tenant declares its part with a `.res` in its file:

* **`GOLD_BUF`**, the inn's gold window's two middle rows of tiles, with
  the party's gold composed into them: `gold_window.s`. It lives from a
  message box's opening to its rows going up to VRAM.
* **`POSED_REG`** and its scratch, **`POSED_T`**, the runs of tilemap
  words of the events drawn into the background: `posed.s`. `build_oam`
  fills them and the vblank sends them up, so they live less than a
  frame; `posed_drain` looks at them only on a map that has such events.

Its size is the converter's —each tenant's part, in `materials.rs`,
which the cartridge's `.inc` carries and each tenant `.assert`s its `.res`
against—, `GRIDHI` takes the rest of the half, which is what puts
`GRID_TOP` at the top, and `far_ram_game.s` asserts that it is.
`test_wram_reservations.py` names every tenant. **What may live there is
what lives less than a map and is not needed after a battle**: the
battle composes its scene over the whole half and nothing puts the region
back, so each tenant rebuilds itself —the gold window composes its rows
again on the next box, and `posed_sync` empties the runs on the way back—.
A new tenant is a `.res` in `GRIDTOP` in its file, a part in
`materials.rs`, a line in `far_ram.s`'s list, a name in that test, and a
line here. Both came from `FARBSS` on 2026-11-16, when A Blurred Line had
used it up.

## What a battle borrows, and what comes back

A battle borrows the upper halves: the troop's battlers go to the grid's
half of `$7E` (`SCN_BATTLERS`, which is `GRID`), the clean background to
the whole of `LOADBUF` (`SCN_CLEAN`, which is `LZ2_BUFFER`) and the
composed scene to the whole of `GRIDHI` (`SCN_SCENE`, which is
`GRID_HIGH`), over the grid and everything behind it. Each is named after
what it borrows, with an `.assert` that it fits, and so is a full
screen's image (`SCREEN_BUFFER`, the whole of `$7F`). Nothing of the map runs while the
battle lasts —its intro composes the scene with the map stopped—, and
`back_from_battle` puts it back: `lz2_grid` decompresses the grid and
what lives behind it again, the bytecode and the page table included;
`posed_sync` writes the drawn-in events' cells back into the grid and
empties their runs —`GRID_TOP` is not put back: its tenants rebuild
themselves—; and `back_to_map` uploads the tiles through
`LZ2_BUFFER` and decompresses the carried bytecode again. What is in
`FARBSS` and the `BSS` is not touched by the battle unless it is the
battle's own.

The same happens, smaller, with every screen that goes through
`LZ2_BUFFER` —the menu, the shop, the save slots—: `back_to_map`
rebuilds what lived there.

## Where it came from

The map of far RAM was written by hand, address by address, until
2026-10-16, when the linker took the whole of `$7E2000`–`$7E7FFF`; how it
was that day, region by region and game by game, is in [The WRAM and SRAM
map, measured](../research/2026-10-15-the-wram-and-sram-map.md). The grid
went to WRAM on 2026-09-26, the block table and the bytecode behind it on
2026-10-01, the page table on 2026-11-16, and `GRID_TOP`'s tenants, which
lived in `FARBSS` until A Blurred Line's ran out of it, the same day. The
upper halves, `GRID_TOP` and the stack were addresses written by hand
until 2026-11-19, when they became areas of the `.cfg` too, at the same
addresses: the task
[`all-wram-placed-by-the-linker`](https://github.com/lautarodragan/the-advocate-work/blob/main/tasks/all-wram-placed-by-the-linker.md).
