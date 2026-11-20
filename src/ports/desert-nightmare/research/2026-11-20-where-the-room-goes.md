# Where the room goes (2026-11-20)

> **Measured on 2026-11-20** by Carpintero, on Desert Nightmare's and
> Vampires Dawn's `release` built from the-advocate-ports `1177cbc` with
> the tools of the-advocate `16081af9`, and on A Blurred Line's `release`
> as the ports' checkout had it built that day. It is a snapshot: the
> numbers are from that day and this file is not updated. How each one
> was obtained is at the end, in "How it was measured". The task is
> [`desert-nightmare-the-cartridge-has-no-room-left`](https://github.com/lautarodragan/the-advocate-work/blob/main/tasks/desert-nightmare-the-cartridge-has-no-room-left.md).

Making the event compiler read the hero's map id made common event 14
compile, its copies into 181 maps stopped the build —"a piece of 23040
bytes needs a new bank, and the 126 banks the cartridge has for data are
taken"—, and the project kept it from being copied, `[events]
commons_not_copied = [14]`. taro asked for the next step to be not needing
that option, and for the options: the ROM's graphics to the `.msu`, or the
painted maps streamed from it in chunks. This is what fills the ROM, what
the copy costs and why, and the options ranked.

## Summary

* **The ROM is not full of what the `.msu` can take.** `rm2k-snes-space`
  says 126 of 126 banks because the packing (`pack_with_msu`) sends to the
  `.msu` **the fewest pieces that make the rest fit**, so a game that
  requires the MSU-1 always looks full. Of what the `.msu` already knows
  how to read, **2,059,635 bytes (31.4 banks) are still in the ROM**: the
  maps' LZ2 streams, the pictures, the samples. What is short is room for
  what **cannot** go there.
* **The copies made the bytecode something that cannot go.** Copied, common
  14 is 7,317 bytes in each map and common 12, already copied, 13,833: most
  of each is a common event it calls, **pasted seven times**. Together they
  put all 181 maps over what a map's bytecode may decompress into WRAM
  (`LZ2_CAP_CODE`, 19,968 bytes), and a map over it keeps its bytecode in
  the ROM **uncompressed**: 3.94 MB of raw bytecode against the 2.06 MB the
  packing could make room for.
* **Common 12's copies draw nothing.** A copy exists so an automatic
  common draws its small pictures with the map's; every picture of 12, and
  of the 11 it calls, is 320x240. Its 181 copies are **about 1.05 MB of
  LZ2, 95% of all the maps' bytecode**; without any copy the maps' pages
  are 57,675 bytes.
* **The recommendation, which makes `commons_not_copied` unnecessary**:
  copy a common only if the copy draws a small picture the shared body
  does not. For Desert Nightmare that drops 12's copies and keeps 14's:
  **352 KB less than today**, the largest map's pages at 10,021 bytes, and
  14's cursor drawn as a copy draws it. Then, in this order: identical
  copies stored once (about 0.69 MB more), and the painted cells stored
  once (**2.4 MB**), which would let the game go back to the MSU-1
  optional.
* **Streaming the painted cells from the `.msu` works on paper and is the
  most expensive**: a walking frame needs 240 to 504 bytes, 2 KB at the
  fastest speed —the same DMA the ROM does today—, and the main loop leaves
  206 of 262 lines free at the median while walking and 67 at the worst,
  measured on map 172; what costs is one seek
  per crossing, which no core of the suite delays, and a game tied to the
  MSU-1 for good. It frees 4.2 MB that the options above make unnecessary.

## 1. What fills the ROM

Every label of the data banks in the build's `.dbg`, sized by the next
label of its bank and grouped by what it is. The data banks hold
7,663,276 bytes —126 banks, 17 of them halves—, and the runtime's code
122,736 more in its own.

| What | Bytes | Banks | Share | Read |
|---|---:|---:|---:|---|
| painted cells, their attributes and column tables | 4,196,264 | 64.0 | 54.8% | a column at a time, while walking |
| maps' bytecode, LZ2 | 995,603 | 15.2 | 13.0% | on entry; may go to the `.msu` |
| maps' graphics streams: tiles, upper layer, sprites, ChipSet animation, grid | 611,242 | 9.3 | 8.0% | on entry; may go |
| poses | 515,320 | 7.9 | 6.7% | every frame |
| precomposed fonts: messages, screens, slots | 364,416 | 5.6 | 4.8% | a letter at a time |
| pictures: tiles and tilemaps | 245,498 | 3.7 | 3.2% | when shown; may go |
| pictures: palettes, tables, the common sets | 97,802 | 1.5 | 1.3% | when shown |
| samples | 198,099 | 3.0 | 2.6% | on a package load; may go |
| maps' tables and palettes | 132,146 | 2.0 | 1.7% | on entry, some every frame |
| texts and faces | 75,472 | 1.2 | 1.0% | a message at a time |
| title, windows, menus, screens | 108,972 | 1.7 | 1.4% | when a screen opens |
| battle animations | 49,201 | 0.8 | 0.6% | in battle |
| panorama | 18,793 | 0.3 | 0.2% | on entry; its tiles may go |
| shared common events | 18,536 | 0.3 | 0.2% | while playing |
| sound effects' tables | 16,804 | 0.3 | 0.2% | on a package load |
| the rest, 291 small kinds | 19,108 | 0.3 | 0.2% | |

And in the `.msu`, the 105 pieces the packing sent there, 1,037,034
bytes: maps' tiles (445 KB), pictures' tiles (263 KB), maps' bytecode
(172 KB), the debug screen (145 KB, which goes in any game) and panorama
tiles (16 KB).

**Half the ROM is the painted maps**: 93 maps, 126,756 cells of 32 bytes,
each map carrying every cell of itself column by column so that a column
goes up in one DMA. **34.7% of those cells are distinct within their
map** (43,935, 1.41 MB), and 41,569 across maps; Cardón measured 29.5%
over 135 maps the day before ([Can the towns be
baked?](2026-11-19-can-the-towns-be-baked.md), section 3).

**The maps' bytecode is a seventh, and almost all of it is common 12's
copies** (section 3). The poses, the fonts and what every screen draws are
the rest of what may not go.

**What may go and stays**: the kinds marked `.msu` by their emitter add
up to 2,059,635 bytes in the ROM. That is the room the cartridge has for
anything that may not go, at the price of what the `.msu` costs a map
load: one to eight frames more in snes9x, and one or two per seek in
Mednafen ("What does not fit in the ROM", in [the `.msu`
file](../../../internals/the-msu-file.md#what-does-not-fit-in-the-rom)).
Vampires Dawn is the same: 126 of 126, and 3,103,859 bytes (47.4 banks)
of what may go still in its ROM.

## 2. What else could go to the `.msu`

The rule of the `.msu` holds: what is read when a map, a battle or a
screen loads, never every frame. These kinds are read that way and are
not marked today:

| Kind | Desert Nightmare | Vampires Dawn | When it is read |
|---|---:|---:|---|
| faces, `message_face` | 29,550 | 104,016 | when a message with a face opens |
| maps' palettes, `*_cgram` | 60,226 | 127,298 | on entry |
| pictures' palettes, `big_colors` | 42,784 | — | when a big picture is shown |
| battle animations' cells, `anim_battle` | 44,765 | 100,470 | when the animation plays |
| the common pictures' sets, `picture_set_tiles` | 17,440 | — | when a set goes up, a few blocks a frame |
| title, `title_lz` | 25,169 | 27,869 | once, after the boot's check |
| windows, menus and screens' graphics | about 80,000 | about 150,000 | when the screen opens |
| Game Over, `game_over_lz` | — | 35,721 | once, when the game is lost |

About **300 KB (4.6 banks) in Desert Nightmare** and **550 KB in Vampires
Dawn**. Each is a small change in its emitter (`.msu` before its `.block`)
and a reader that knows the port: the LZ2 ones already go through
`lz2_decompress`, the raw ones need `msu_dma0` or a copy loop. What it
costs is one more seek where it is read —a frame or two in Mednafen when a
face opens—. Not worth it before the copies, which free more for less, and
none of it would be needed for the commons.

What may **not** go, and why: the poses (every frame, see [poses are
uploaded on the fly](../../../optimizations/lossless/characters-and-sprites/poses-on-the-fly.md)),
the painted cells (a column while walking: section 4), the fonts (a
letter while a message types; the debug screen does it with a cache of
letters and a seek per miss, which a message cannot afford in Mednafen),
and what the CPU reads while playing —the shared bodies, the texts, the
tables—. The maps' bytecode is already a candidate when it is LZ2: it is
read once, on entry. **What made it stay is the WRAM cap**, not the
`.msu`: a map over `LZ2_CAP_CODE` keeps its bytecode in the ROM raw, and
there it is read while playing.

The fonts are three copies of one: `font_pre`, `scr_pre` and `slot_pre`
are the game's letters precomposed at eight phases for three palettes,
364,416 bytes in Desert Nightmare and in A Blurred Line, 376,896 in
Vampires Dawn (draft
[`the-font-is-precomposed-three-times`](https://github.com/lautarodragan/the-advocate-work/blob/main/tasks/the-font-is-precomposed-three-times.md)).

## 3. Why common 14 costs 7 KB a map, and common 12 14 KB

A copy is the automatic common compiled as a page of a last event of each
map —`common_pages.rs` in the converter—, so its pictures are the map's;
the shared body starts nowhere there. The converter copies every automatic
common with a `ShowPicture` that something can turn on, into every map
with room for one more event: Desert Nightmare's 12 and 14, into 181 of
its 189 maps.

Compiled by the event compiler with the request the build leaves, with
14 copied and without:

| | Shared body | A copy | The copy without its `CallEvent`s |
|---|---:|---:|---:|
| common 14 (maps screen) | 3,207 | 7,317 | 3,243 |
| common 12 (notes screen) | 689 | 13,833 | 1,023 |

**A copy pastes what it calls.** 14 calls common 13 seven times and 12
calls 11 seven times; a page calls a shared body only when the body
compiled with the map is byte for byte the one without it (`shares` in the
event compiler), and 11 and 13 show pictures, which compile against the
map. So each copy carries seven copies of its callee: 4,074 bytes of 14's,
12,810 of 12's.

**What the copies draw**: every picture of 11, 12 and 13 is 320x240 —the
notes, the inventory, the maps—, which the console does not show in
either path ("its 96 big pictures are not shown", the converter log says
of 12). 14's only small picture is its cursor, `Kartencursor`, 7x4. So
**12's copies draw nothing** (draft
[`desert-nightmare-common-12-is-copied-and-draws-nothing`](https://github.com/lautarodragan/the-advocate-work/blob/main/tasks/desert-nightmare-common-12-is-copied-and-draws-nothing.md)),
and 14's draw the cursor. Shared, the cursor goes in the cartridge's table
of common pictures, in one set with the two 320x84 textbox pictures, 273
tiles, which fits beside the map's bank on **10 of the 189 maps** (draft
[`desert-nightmare-the-map-cursor-is-shared-on-ten-maps`](https://github.com/lautarodragan/the-advocate-work/blob/main/tasks/desert-nightmare-the-map-cursor-is-shared-on-ten-maps.md)).
The rule that decides the copies is from 2026-10-17; the shared table,
from three days later.

**What each choice costs**, the maps' pages summed and compressed with
`lz2` (pages only: the routes and the page tables add the same to each):

| Copied | Raw | LZ2 | Maps over `LZ2_CAP_CODE` |
|---|---:|---:|---:|
| none | 112,380 | 57,675 | 0 |
| 12 (today) | 2,616,153 | 1,106,568 | 0 |
| 14 | 1,436,757 | 754,186 | 0 |
| 12 and 14 | 3,940,530 | 1,754,389 | 181 |

**And the copies are the same in every map**: 14's copy is one bytecode,
byte for byte the same in the 181 maps, and so is 12's. Each map carries
its own (draft
[`identical-copies-of-a-common-event-are-stored-in-every-map`](https://github.com/lautarodragan/the-advocate-work/blob/main/tasks/identical-copies-of-a-common-event-are-stored-in-every-map.md)).

So, to the four questions:

* **Shared**: identical copies can be one block in the ROM that each map's
  page points to. The page table already takes a pointer into the ROM —a
  map over the cap does exactly that (`code_in_rom` in
  rm2k-snes-mapdata), and the interpreter, the routes and the battle
  animation take the bank from the pointer—; what is missing is a table in
  WRAM pointing to a block the linker places, since a compressed table
  carries numbers (`pages_in_wram`), and the block not crossing a bank.
  14's copies would cost 7,317 bytes once instead of about 700 KB of LZ2.
* **Deduplicated**: the same thing, by content: the converter already
  shares two maps' identical ROM code blocks; the copy would have to be its
  own block.
* **Smaller**: the seven pasted callees are most of each copy. 11 and 13
  show only 320x240 pictures; if `shares` took a picture neither path
  shows as the same —and nothing else of theirs changes with a map, which
  was not checked—, the copies would call the shared bodies: 14's copy
  3,243 bytes, 12's 1,023.
* **From the `.msu` on map entry**: it already is, when it fits: a map's
  bytecode in LZ2 is one of the pieces that may go. The copies break the
  WRAM cap, not the ROM, and that the `.msu` does not change.
* **Not copied**: right for 12, which draws nothing. For 14 it loses the
  cursor on 179 maps, which is what `commons_not_copied = [14]` does
  today.

## 4. Streaming the painted cells from the `.msu`

### The port's speed

The data port is read by DMA with the source fixed at `$2001`, 8 master
cycles a byte, which is any DMA's speed: **the same as from the ROM**, 170
bytes a scanline. And it can go straight to VRAM: the DMA's A-bus side
is `$2001` and the B-bus side `$2118`, as from the ROM. So bandwidth is
not the question; **the seek is**.

* **snes9x, bsnes and Mesen2**, the harness's cores, seek instantly —
  Mesen2's `_dataBusy` is "always false" in its `Msu1.h`—. No test of the
  suite can see a seek take time.
* **Mednafen** takes one or two frames per seek ([the `.msu`
  file](../../../internals/the-msu-file.md#how-long-it-takes)).
* **The FX Pak** keeps 16 KB of the file in two halves of 8 KB
  (`MSU_DATA_BUFSIZE` in its firmware's `src/msu1.h`): a seek inside either
  half only moves the address; outside them, `prepare_data` in its
  `src/msu1.c` seeks the file and reads 16 KB from the card, with the data
  busy bit up. Reading on past a half makes the firmware refill it with the
  next 8 KB, without the busy bit. A seek with a track playing was measured
  at about 2 ms ([after a save](../../../internals/the-msu-file.md#after-a-save-a-seek-nobody-reads)): an eighth of a frame.
  The rate of the card was not measured.

### What a walking frame needs

A painted map's screen is 33 columns by 30 rows of 8x8 cells in VRAM
(`PAINT_COLUMNS`, `PAINT_ROWS`), and a crossing uploads a column of 30
cells or a row of 33, 32 bytes each: **960 and 1,056 bytes**. The walk is
two pixels a frame, a crossing every four frames (`painted.s` spreads a
row over three, `PAINT_STEP`):

| Speed | A column | A row | Diagonal | DMA time |
|---|---:|---:|---:|---:|
| walking, 2 px a frame | 240 B a frame | 264 | 504 | 3 lines |
| 4 px a frame | 480 | 528 | 1,008 | 6 lines |
| 8 px a frame, the fastest | 960 | 1,056 | 2,016 | 12 lines |

Averaged over the frames: the column goes in the frame of its crossing,
and the row over three. Their attributes, a byte a cell, are 3% more and
can stay in the ROM.

### The frame's room while walking

Mesen2's core with the harness's line profiler, on map 172 (`Alte
Minensiedlung`, painted, 40x28), entered like a door at its start and
walked in the four directions and the four diagonals for 1,030 frames, of
which the camera moved on 360 —38 column crossings and 52 row crossings;
the hero runs into walls the rest—. Free is what is left of the frame's
262 lines when the main loop gets to `wait_vblank`, counted from the NMI:

| Frames | How many | Free lines, median | Free lines, worst |
|---|---:|---:|---:|
| every one | 1,029 | 166 | 61 |
| the camera moving | 360 | 206 | 67 |
| a crossing | 90 | 120 | 67 |
| a row crossing | 52 | 107 | 67 |

No tick ran past its frame. On map 4, the workshop (22x19), with fewer
crossings, 219 at the median and 74 at the worst. **Even the worst
crossing frame has 67 lines**, about 4 ms, against the 3 to 12 lines of
DMA a strip takes and the four register writes of a seek; and the strip's
DMA is the one the ROM's path already does in the vblank today. So the
CPU and the bus have room; the only time that is not ours is the seek.

### A design

Not built; what it would take:

* **The cells twice in the `.msu`**: by columns, each the map's whole
  height, and by rows, each its whole width. A column crossing is one seek
  to `columns[x] + 32 × top` and 960 bytes; a row crossing, one seek to
  `rows[y] + 32 × left` and 1,056. Walking straight, the next seek lands
  within the FX Pak's 16 KB nearly always —a column of a 30-row map is 960
  bytes apart from the next—; turning or walking diagonally alternates the
  two regions, and each change is a read of the card.
* **The seek is asked a frame before it is read**, right after the
  previous strip's DMA, toward the column or row the camera is going to;
  in the vblank, if the busy bit is down, the strip goes from `$2001`
  straight to its VRAM slots —a column in one or two DMAs, a row in 33—,
  and if it is up, it waits a frame and the screen's edge shows the cell
  that left for one frame, which is what a late queue shows today.
* **WRAM**: none for the cells, if they go straight to VRAM; a strip's
  staging buffer of 1,056 bytes, 2,112 with two, if the read is to be done
  outside the vblank. The far RAM has 2,631 bytes free today.
* **Every reader seeks**: a big picture shown from the `.msu`, a
  `ChangePBG`'s panorama, an audio package's effects, the debug screen's
  letters move the port; the stream re-seeks after any of them because it
  always seeks before it reads.
* **The entry**: `paint_load` uploads the 33 columns with the screen off;
  by columns they are one range of the file, so one seek and a read
  through it, skipping the rows outside the window.

**What it frees**: the 4,196,264 bytes of painted cells and tables, minus
nothing; the `.msu` grows by twice the cells, 8.1 MB, which nothing limits.
**What it costs**: `painted.s`'s queue and flush rewritten around the port;
the converter's paint tables pointing into the `.msu`; and a latency path
that no core of the suite runs, so it would be verified by hand in Mednafen
and on the console. **And the game requires the MSU-1 for good**: the
cells would be only there.

## The options, ranked

| | Option | Cost | Room in Desert Nightmare | Needs the MSU-1 |
|---|---|---|---:|---|
| 1 | **Copy a common only if the copy draws a small picture the shared body does not** | the converter, small | 352 KB (12's copies out, 14's in); 1.05 MB with neither | no |
| 2 | Identical copies stored once in the ROM | the converter and the page table, moderate | about 690 KB more | no |
| 3 | A copy calls the shared bodies whose difference with a map is not shown | the event compiler, small to moderate | 14's copies to 44%; overlaps 2 | no |
| 4 | **The painted cells stored once**, gathered by an index | the converter and `painted.s`, moderate | about 2.4 MB | no |
| 5 | More kinds to the `.msu`: faces, palettes, title, screens, battle animations | each emitter, small each | about 300 KB | yes |
| 6 | The font precomposed once | the runtime's text, moderate | about 240 KB | no |
| 7 | The painted cells streamed from the `.msu` | `painted.s` and the converter, large; a path no core runs | 4.2 MB | yes, for good |

The room is not additive between 2 and 3, and 7 replaces 4.

## The recommendation

**Option 1 first, and alone it is enough**: the copy rule asks today
"does the common show a picture?"; it should ask "does the copy draw on
this map a small picture the shared body cannot?" —big pictures never,
small ones only where the common table's set does not fit—. For Desert
Nightmare that copies 14 and not 12, and the cartridge has 352 KB more
than today with the cursor still drawn; `commons_not_copied` stops being
needed and can go. And the converter should weigh a copy against
`LZ2_CAP_CODE` instead of letting a map's bytecode fall into the ROM raw:
a copy that does not fit should not be made, and should say so.

**Then option 4**, the draft
[`painted-maps-carry-every-cell`](https://github.com/lautarodragan/the-advocate-work/blob/main/tasks/painted-maps-carry-every-cell.md):
2.4 MB, in the ROM, for any game with painted maps. With 1 and 4, Desert
Nightmare's data would be about 5.8 MB of the 7.66 the banks hold, the
`.msu`'s pieces included and the debug screen's left there: **its MSU-1 could be optional again**, which is
what [the decision](../decisions/the-msu-1-is-required-for-now.md) asks
for when the cells are stored once. That was not built, so it is an
estimate.

**Option 7 is not needed** by Desert Nightmare after 1 and 4, and by no
other game today: it is the way only if a game's painted cells, stored
once, still do not fit.

**Vampires Dawn**: it copies no common and paints no map, so 1, 2, 3, 4
and 7 change nothing in it. It is not out of room either —3.1 MB of what
may go is still in its ROM—; what fills what may not go is its music
streams (1.12 MB) and its poses (1.21 MB). Option 5 would give it about
550 KB, and 6 about 250 KB.

**A Blurred Line**: copies nothing and paints nothing; nothing of 1–4
changes it. Its `.msu` stays optional, so 5 and 7, which move data that
then exists only in the `.msu`, do not apply to it; 6 would give it
about 240 KB like any game.

## How it was measured

* **The builds**: `just cartridges` in `desert-nightmare/` and `just
  cartridges release` in `vampires-dawn/` of a worktree of the ports at
  `1177cbc`, with the tools of `16081af9` installed; A Blurred Line's
  `build/release/abl.sfc.dbg` as the ports' checkout had it.
* **What fills the ROM**: every `sym` of type `lab` in a `BANK*` segment
  of the `.dbg`, sized by the next label of its segment, grouped by its
  name without the numbers; the `.msu`'s pieces are the bank-`$3D`
  constants the packing leaves, sized by the next unit. A throwaway
  script, as is each below.
* **The painted cells**: the `paint_tiles_*` blocks read from the ROM by
  their `.sym` addresses, cut in 32 bytes, and counted distinct per map
  and over all.
* **The copies**: `rm2k-snes-events compile` with the build's
  `events-request.json`, its `common_pages.commons` set to `[]`, `[12]`,
  `[14]` and `[14, 12]`, and with a copy of the dump whose commons 12 and
  14 have no `CallEvent`; each map's pages' `code` concatenated and
  compressed with `lz2 compress`.
* **The common set that fits**: byte 5 of each 8-byte entry of
  `picture_common_maps` in the ROM.
* **The frame**: the ports' harness `Session` on Mesen2's core, the title
  passed and control reached, `go_to_map` to map 172 at its start, control
  again, and then the buttons held frame by frame with Mesen2's watch on
  `wait_vblank`: the line it was hit on each frame, counted from the NMI's
  line, and `camera_x`/`camera_y` read every frame to tell the frames the
  camera moved and crossed a cell.
* **The FX Pak**: read from its firmware's source, `src/msu1.c` and
  `src/msu1.h`, not measured.
