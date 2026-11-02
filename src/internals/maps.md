# Maps

#### The map is not stored as a tilemap

Expanded, it would not fit in the cartridge: 1000×80 cells at two bytes
each are 156 KB, more than the whole ROM. What is stored is **RM2k's 16×16
cell**, which is the unit the game was drawn with and which repeats a lot.

```text
map 52, 1000x80 cells    expanded tilemap   160,000 B
                         block grid          40,000 B + 1,744 B of table
```

A block carries its four background cells, its four cells of the layer
above, and its passability, all together. In the whole game the maximum is
512 distinct blocks.

Having passability travel in the block **erased the whole collision
table** —one byte per cell, up to 20 KB—. And it is not a shortcut: two
cells that are drawn the same could have different permissions if they came
from different ChipSet tiles, so passability goes into the key blocks are
deduplicated by, and those two separate on their own.

Where the hero is inside the grid isn't an index either but a **pointer
that moves with him**, one cell or one row per step. That way nothing ever
has to multiply or build a 24-bit address from scratch, which is awkward
because the grid crosses banks and the banks are not contiguous: going past
`$FFFF` leads to the next bank's `$8000`, and only `ptr_more` and
`ptr_minus` know that.

#### The grid lives in WRAM, and the cartridge carries it compressed

Since 2026-09-26 the grid travels in the cartridge as LZ2 and is
decompressed whole when the map loads, at `$7E8000`, continuing at
`$7F8000` if it is over 32 KB (`GRID` in `runtime/far_ram.s`,
`lz2_grid` in `decompress.s`). The upper halves of two consecutive WRAM
banks have the shape of two LoROM banks, so the readers didn't change:
the pointer used to point into the cartridge, now it points into WRAM. A
battle overwrites the `$7F` half, and `back_from_battle` decompresses it
again. What reserving that RAM cost and how it is reverted:
[the grid lives in WRAM](../optimizations/lossless/the-map/the-grid-in-wram.md).

**A cell is one byte in a map with at most 256 blocks**, and two in the
others, since 2026-10-17. Most maps have that few —305 of A Blurred Line's
313 that day—, and narrow is not kept for the maps that need it: every map
that can goes narrow, so the suite plays that path on every map and not on
one. The record says it in the fourth byte of the grid's pointer
(`M_GRID_NARROW`), which a pointer does not use; `enter_map` turns it into
`cell_bytes` and `cell_mask`, and whoever reads a cell masks it with the
second and whoever steps over the grid steps the first. The 64 KB then hold
65,536 cells less what lives behind the grid, instead of 32,768 less half
of it: Zounds' dream, 200x200, is 40,000 bytes. A map that fits neither way
is left out of the cartridge with a warning and a `map-left-out` in
`degradations.json`, and a teleport to it is the box that says the map is
missing. Which maps go narrow and the limits: `cell_bytes` in
`materials.rs` of `rm2k-snes/mapdata/`.

The same happens with the tiles of the background, of the layer above, of
the sprites, and with the drawings of the ChipSet animation: they travel as
LZ2, are decompressed to `$7F0000` and uploaded with the same DMA as
before. The animation drawings stay at `$7F6000` for as long as the map
lasts, because they are uploaded every time the clock changes. How each
block is split into streams and with which WRAM limits: `materials.rs` in
`rm2k-snes/mapdata/`; the format and the compressor:
`snes/lz2/`, which the emission uses as a library.

And since 2026-10-01, **the block table, its passability and the event
bytecode**: the CPU reads them while the game is played, so they are not
uploaded to VRAM but live in WRAM **right behind the grid**, in the `$7F`
half. The blocks travel as a third stream stuck to the two of the grid, and
the bytecode in one of its own, which the `map_code` table names;
`lz2_grid` decompresses each where the previous one ended, and they come
back with it after a battle. What happens to a page that keeps running in
another map: see [Events](events.md). The address is not computed by the
ROM: the converter puts it in the map's record (`map_ram_pointers` in
`record.rs`), and the test below checks that it matches where things ended
up. The converter stops if a map doesn't fit: what the grid takes of `$7F`
plus what goes behind it can't exceed its 32 KB. The bytecode is the
exception: a map whose code does not fit keeps it in the cartridge (see
[Events](events.md)).

What is left in VRAM and WRAM after loading is compared by
`tests/rpgmaker/test_decompressed_load.py` in the ABL port against what
the converter leaves uncompressed in ABL's `build/<cartridge>/crudos/`.

#### What goes in the layer above

A tile RM2k draws above the hero —a lower tile with the "above" or wall
bit, an upper tile with "above"— goes to the layer above, BG2, so the hero
and the events walk behind it. **Unless no character can ever be drawn
under it**: since 2026-11-02 the tiles above the hero of a cell no sprite
can cover are drawn into the background, over the cell's own tiles below,
as RM2k stacks them. The map looks the same, pixel for pixel; what changes
is that the window without a background, which takes BG2 on its lines,
no longer blacks those cells out —see [The window without a background
does not cover the characters](../divergences/the-window-without-a-background-does-not-cover-the-characters.md)—,
and that BG2 carries less.

**Only over nothing**: a cell goes below only if what shows through its
tile above is black or the panorama's hole —a cave's ceiling, a wall
drawn only on top—. There the background's new cell is the tile above
alone, so there are as many as the layer above had, and as many blocks.
Over something drawn —a roof's edge over grass— every pair would be a
cell of its own, 8-bit, and on 2026-11-02 that took A Blurred Line past
its cartridge. It is also where it shows: over nothing the window leaves
the cell black, over grass it leaves grass.

**And only if the map loses nothing for it.** The tiles move to the
background with their colours, and a background that was full would pay
for them elsewhere —merging colours, or fusing cells to fit its layout—:
on 2026-11-02 that was 36 of Vampires Dawn's maps. So Wally converts the
map both ways and keeps the lowered one only if its panorama goes the same
way, neither layer merges more colours and no more cells are fused
(`lowering_costs` in `map.rs`); if not, the log says why they stay above.

**A project asks for it**, with `[graphics] unreachable_above_below =
true` in its `advocate.toml`: what goes below moves from the 4-bit layer
above to the 8-bit background, a few more bytes per map, and a cartridge
with no bank left —A Blurred Line's release, that day— does not take them.
Without the key the converter sends Wally no `reach` and every tile above
the hero goes above, as before.

**Which cells** is RM2k's question, and rm2k-rules answers it,
`reach::covered` in the-advocate: a character's 24x32 frame on cell (x, y)
covers 4 px of x−1 and x+1 and all of y−1, so the cells to keep above are
those beside or above a cell where some character can be, at some moment.
Where a character can be is a fixed point over everything that can move
one on the map: the hero walks every cell passable from some side from
wherever he arrives; an event walks wherever its movement type takes it;
and every move route that can run there —a page's own, a `MoveEvent` of a
page, of a common event or of a test page— is walked from every cell its
character can be on, through walls if it turns that on, jumping where it
jumps —a jump lands on a cell passable from some side, and rises a row
more—. A page that starts by the action key or a touch walks the hero from
beside its event, which is what keeps a door that walks him through a wall
from walking him through every wall.

**Where the hero arrives** only the whole cartridge knows: the converter
gathers it once (`reach.rs` in rm2k-snes-convert) —the game's start, every
`Teleport`, teleport and escape target into the map, the vehicles— and
sends it in each map's Wally request as `reach`. Wally's background half
draws the cells (`layers::below` and `lower_cells`); the characters half
knows nothing of it.

**What cannot be proved stays above**, and some things put a character
anywhere, which keeps the whole map above: a `ChangeEventLocation` by
variables, a route that leaves walking through walls on for someone who
then walks on his own, an airship. A cell with a tile a `TileSubstitution`
names stays above too, because its block is swapped by tile. **The one
thing taken on trust** is a `RecallToLocation` to a place the game works
out —ABL recalls the hero's cell five rows up—: it is taken to be a cell
one can step on, of any map.

The converter's log says, map by map, `N cells draw their tiles above the
hero below`. What it does not follow is variables: Zounds' map 96 keeps
its whole ceiling above because a statue there is pushed through walls
only when its own coordinates, copied to two variables, say so.

#### A map that loops

A map with RM2k's `scroll_type` loops on that axis —a torus: the hero,
the events and the camera go across its edge to the other side—, and since
2026-10-19 the console does it too, in `runtime/wrap.s` of the-advocate.
What RM2k does and which maps of the ports use it:
[Maps that wrap around](../research/2026-10-19-maps-that-wrap.md).

**The record says which axes** in the fourth byte of its last pointer, the
ChipSet animation's (`M_WRAP`: bit 0 across, bit 1 down), as the grid's
says whether it is narrow; the converter decides it (`wrap_axes` in
`emit.rs` of rm2k-snes-convert). An axis does not loop on a map narrower
than 20 cells across or 15 down, nor on a painted map —none of the ports'
is either—, and such a map says `map-does-not-wrap` in
`degradations.json`.

**The camera becomes a continuous view.** The window keeps cell (x, y) in
slot (x mod 64, y mod 32), and a map's width is not a multiple of 64, so a
camera that jumped by a lap at the seam would move every slot with it. On a
looping axis `camera_x` —what the window and the scroll registers follow—
moves by the change of the camera on the map, the short way round, and is
kept between 2048 and 10240 by jumps of 4096, a multiple of the window,
with `win_col` moved the same. Whatever subtracts the camera from a
position —the characters in OAM, the battle animation over one, the
message that dodges the hero, the walkers' "in sight"— folds the result
into one lap of the map around the screen. That is also why a looping axis
needs 20 cells by 15: a lap wider than the screen by a character on each
side never shows one on both sides, so RM2k's clones are not needed.

**The strips read the map modulo its size**: on a looping map
`build_column` and `build_row` are `wrap.s`'s, which compute the map's
column of a window column from the view and point into the grid again at
the seam. The pointer that moves with the hero is computed again when his
step crosses a seam —his straight step across goes through `cell_allowed`,
as his diagonal does—, and so is the one a step checks for an event.

**All of it is assembled only into a cartridge with a map that loops**, in
`FAR3`, called with `jsl` from hooks in bank 0, and a cartridge without one
—A Blurred Line's, The Book of Three's— comes out byte for byte as before.
The hooks that do the whole of a routine return from it without a branch
of their own, which is what bank 0 can afford: `RTS0` in `wrap.s` takes the
`jsl`'s three bytes off and jumps to an `rts` in bank 0. The other way,
`CALL0` calls a bank 0 routine that ends in `rts` by pushing where that
`rts` goes —an `rtl` in bank 0— and where the `rtl` comes back to, and
jumping. Both are plain `jml`s, and Goombario follows both paths.

What it does not do, because no map of the ports needs it: clones of a
character on both sides of a seam, the panorama's rule on a looping axis, a
jump across a seam, the action key and an event's bump on the hero across a
seam, and the order of the characters across a horizontal seam.
