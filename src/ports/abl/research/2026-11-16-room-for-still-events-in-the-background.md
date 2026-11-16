# Room for still events in the background

> **Measured on 2026-11-16**, by Mulita, with the tools at `a83c4a12`
> (the-advocate) and ABL at `ff7ac5b` (the-advocate-ports), on `release`.
> It is a snapshot: the numbers are from that day and this file is not
> updated. What a cartridge has left **today** is said by `just space`.
> How each number was obtained is at the end. The task is
> [`abl-room-for-still-events-in-the-background`](https://github.com/lautarodragan/the-advocate-work/blob/main/tasks/abl-room-for-still-events-in-the-background.md).

The question: [still events drawn into the
background](../../../divergences/still-events-drawn-into-the-background.md)
—`[graphics] posed_events`— on every map of A Blurred Line costs
**38,722 bytes** more data (map tiles +32,103, the posed record +15,699,
grids +4,749, against sprite tiles −8,522, poses −2,588, NPC tables
−2,448), and `release` uses 126 of its 126 data banks with 527 bytes of
gaps. taro wants the whole game to have it, and asked whether something
could move to the `.msu` without the game stopping being playable
without it. Nothing here is implemented.

## Summary

* **One lossless change frees three times what is needed: the event page
  records in LZ2, behind the grid.** They are 193,376 bytes stored raw
  and 68,513 in LZ2, piece by piece: **−124,863**. They carry no linker
  symbol —their code pointer is already `GRID_HIGH` plus an offset,
  because the bytecode lives behind the grid—, so the converter can
  compress them, and they fit there: the largest record is 12,192 bytes
  and the most a map takes of that 32 KB today is about 12 KB. Without
  MSU-1 the game is the same game; it works for every port.
* **The next lossless ones**: the panoramas' tilemaps in LZ2 (−100,562,
  if 9,600 bytes of WRAM stay with the map) and their tiles in LZ2
  (−84,911, as in October).
* **The `.msu` has one thing that costs no runtime work**: the ROM's
  copies of the sound effects at a lower rate. With MSU-1 the effects
  already come from the `.msu`; capping the ROM's at 11,025 Hz frees
  **109,077** bytes and only changes how 18 effects sound **without**
  MSU-1 —which is how taro plays, in Mednafen—.
* **The debug screen is 146,168 bytes**, 121,056 of them its own
  precomposed font. `debug_screen = false` frees them (`release` packs
  into 122 banks); a one-bit font for that screen alone would keep it for
  about 7 KB, drawing its letters slower.
* **The posed tiles as an MSU-only improvement are not worth it**: it
  frees nothing —the ROM keeps today's map whole, plus whatever tells it
  the other one exists— and doubles every posed map.
* **No chest goes into the background**, so none of the 38 KB is theirs.
  The chest in `BLObjs.png` stands 1 px right of its cell and 2 px above
  it; drawn one pixel to the left and two lower, 33 of ABL's 101 chests
  would go in, for about 80 bytes on top of the rest.

## 1. Where the bytes are

The data banks of `release`, by kind of label, 7,667,185 bytes —the same
count from the link's `.dbg` and from the converter's `.inc`—:

```text
kind                                              bytes     share
map background: tiles (LZ2)                    1,453,525    19.0%
music: song streams                            1,166,681    15.2%
BRR samples (instruments and effects)            860,308    11.2%
characters: per-map sprite tiles, tables,
  palettes                                       560,767     7.3%
battle and the database (battlers, backdrops,
  animations, troops, items, skills)             530,622     6.9%
characters: poses (raw, without empty tiles)     476,552     6.2%
map background: grids, blocks, colours, the
  layer above, ChipSet animation                 412,626     5.4%
pictures (big, mode 7)                           368,082     4.8%
panoramas: tiles (raw)                           358,720     4.7%
texts: messages                                  212,084     2.8%
map events: bytecode (LZ2) and tables            205,630     2.7%
map events: page records (raw)                   193,376     2.5%
menus, title, save screens                       176,637     2.3%
panoramas: tilemaps (raw)                        171,714     2.2%
debug screen (its font 121,056)                  145,640     1.9%
font precomposed: message window                 126,394     1.6%
font precomposed: text screens                   119,808     1.6%
font precomposed: save slots                     119,808     1.6%
the rest                                           8,211     0.1%
```

And the code, outside the data banks: bank 0 32,262 bytes, bank 1
31,347, `FAR2` 30,278 and `FAR3` 23,915 (`just space` says what each has
left).

## 2. The options

Ranked by what they free per what they cost. "Whole without MSU-1" is
the standing rule of [the `.msu` file](../../../internals/the-msu-file.md).
"Measured" means over the bytes of this `release`; what is an estimate
says so.

| option | frees | cost | whole without MSU-1 | other games |
|---|---:|---|---|---|
| **page records in LZ2, behind the grid** | **124,863** | decompressed at each map load with the grid's stream, ~220 B per map on average; `page_ptr` points into WRAM instead of ROM | yes, lossless | yes: every port has them |
| panorama tilemaps in LZ2, kept in WRAM | 100,562 | 9,600 B of WRAM with the map —behind the grid, where it is not yet checked map by map that they fit with the records— | yes, lossless | yes |
| panorama tiles in LZ2 | 84,911 | decompressed through `LZ2_BUFFER` at load, about 3 frames for the largest (18 KB); a `ChangePBG` mid-map must not clobber what lives in that buffer | yes, lossless | yes |
| effects at 11,025 Hz in the ROM (16 kHz: 56,593) | 109,077 | none with MSU-1; without it 18 effects sound duller | yes, worse sound | yes: a number in the project |
| debug screen off for a delivery | 146,168 | taro loses the debug screen | yes | yes |
| debug screen with a one-bit font | ~114,000 (estimate) | its letters draw about twice as slow; the composer existed and was measured in October | yes | yes |
| the save slots with the text screens' ink layout | 119,808 | `slot_pre` becomes `scr_pre`; the slots' palette has to take the ink in entries 1–13, and today palette 0 holds it and a red one in 12 —not checked that it fits— | yes | yes |
| big pictures' tiles in LZ2 | 41,899 | decompressed when shown | yes | yes |
| the posed record in LZ2, behind the grid | 9,971 | as the page records: lowers posed's own cost to ~28.7 KB | yes | yes |
| songs whose repeats may nest | 124 KB in October, not measured again | a return stack in the music reader | yes | yes |
| posed tiles only with MSU-1 | nothing | see below | yes | yes, at the same cost |

**What is not worth looking at again**: LZ2 for the precomposed fonts
(484 KB, 5:1) —a letter is read at random, and 120 KB per font has no
WRAM to land in—, and LZ2 for the already-compressed families (map
tiles, sprite tiles, grids, battlers), which [Fitting the whole game in 8
MB](2026-10-14-fitting-the-whole-game-in-8-mb.md) already measured.

### The posed tiles only with MSU-1

The `.msu` already carries the battle backdrops in their best form and
the ROM a lighter one, and the effects the same way. For the posed
events the "lighter form" would be today's map, whole, and the `.msu`
would carry a second version of each posed map: its tiles, its grid and
blocks, its sprite bank, its palettes and its posed record —everything
Cardón's measurement saw change—. What the runtime would need: at each
map load, whether the MSU-1 answers and whether this map has a posed
version; if so, every one of those streams from the `.msu` (the readers
exist, `lz2_msu` and `msu_dma0`, from [what does not fit in the
ROM](../../../internals/the-msu-file.md#what-does-not-fit-in-the-rom)),
the event tables told which events are no longer objects, and the
tables that are not streams —NPC poses, actor tables— in two versions or
loaded into WRAM. It frees nothing in the ROM, which still needs today's
map plus a directory of the other one, and it would be the first time a
map plays differently with and without the `.msu`, so the suite —which
runs without MSU-1— would need cartridges of its own to see it. Any of
the lossless rows above is cheaper.

## 3. The chests

ABL has 101 chests: `BLObjs` character 2, closed pose 0 and open pose 1
facing left, and four ChipSet-tile ones on map 17. With `posed_events =
true` none goes into the background:

* **33** (maps 38, 59–62, 261, 263, 289, 292, 293): a pose is drawn past
  its cell. The chest is 16×15 px at x 5–20, y 14–28 of its 24×32 frame,
  so on the map it stands 1 px right of its cell and 2 px above it, and a
  background block only covers the cell.
* **49** (maps 17, 20, 21, 26, 49, 133–140, 316): a command moves them.
  ABL's older chests are two events, a closed one and an open one waiting
  at (0,0), and opening swaps them with `SwapEventLocations`.
* **19** (maps 32 and 255): their panorama stays, and `posed_events`
  skips the map.

With a copy of the game whose two chest frames are moved 1 px left and
2 px down, the 33 go in, and on top of every other map's posed events
they add about 80 bytes. Only those ten maps —their other 65 still events
come along— cost +4,720: a chests-first list fits once there is room,
but not before, and it needs the chest drawn inside its cell (a patch to
the image, or Wally accepting a pose that spills a few pixels).

## How it was measured

* **The inventory**: `release` built with the global tools; each label
  of the data banks in ld65's `.dbg`, up to the next label, grouped by
  its name without the trailing number. Two whole banks carry no label:
  the middle of the text block and the tails of song streams that cross
  banks; they are counted where they belong. The same total comes from
  summing the operands of the converter's `.inc` as the packing does.
* **posed_events**: the same `release` converted with `posed_events =
  true` (the packing fails, the `.inc` stays), compared family by family
  with the one without it. It gives Cardón's 38,722 bytes. With the tools
  of `a83c4a12` installed, map 8 goes in too, for 619 bytes more
  (Cardón's measurement).
* **LZ2**: `lz2 compress --no-increasing`, piece by piece, over the
  bytes read from the ROM at each label's address. Pieces that would be
  compressed together —the records with the grid's stream— were not
  measured together.
* **The effects**: each effect's bytes and rate from the build's
  `audio.json`, scaled by the new rate over the old one.
* **The debug screen**: `release` converted with `debug_screen = false`.
  The one-bit estimate is the font and the shift table the message
  window's one-bit composer had on 2026-10-03, about 6 KB, against
  121,056.
* **The chests**: the game's events in `data/`, by graphic and name, and
  the converter's log of the posed builds, event by event; the edited
  image in a scratch copy of the game, never in the original.
* **Not measured**: the cycles of any new decompression on a console;
  the page records and the panorama tilemaps behind the grid map by map;
  the save slots' palette with the fixed ink layout; the songs again.
