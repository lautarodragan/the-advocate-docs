# The SRAM as slow WRAM

> **Measured on 2026-11-16** by Aguará, with the tools of the-advocate at
> `a7c8c2d0` and the ports at `b024af2` plus the test ROM of branch
> `aguara-sram-dma` (`0731851`). The emulators' and the FX Pak's behaviour
> was read in their sources at these commits: sd2snes `cf7e21d` (firmware
> 1.11.2), snes9x `890b5d44` (the libretro core the harness runs), bsnes
> `591b7e1` and bsnes-libretro `a7e88d6`, ares `4cb8d92`, Mesen2 `b9fa69d`,
> mesen-s `9e4fdeb`, Mednafen 1.32.1. It is a snapshot: the numbers are from
> that day and this file is not updated.

taro's question, on 2026-11-16, in the task
[`the-sram-as-slow-wram`](https://github.com/lautarodragan/the-advocate-work/blob/main/tasks/the-sram-as-slow-wram.md):
whether the part of the SRAM the save slots do not use could serve as
working memory, "to keep WRAM temporarily when we compress / decompress",
between a fade out and a fade in —entering or leaving a battle, changing
maps—; with a 256 KB SRAM, which "seems to be a number the FX Pak and
emulators accept"; and whether the FX Pak writes its card every time the
SRAM changes.

## Summary

* **256 KB is `$FFD8 = $08`, and it is exactly what ExHiROM can address**:
  the SRAM window is `$20-$3F:6000-7FFF` (mirrored at `$A0-$BF`), 32 banks
  of 8 KB. The game already uses that window, a bank per save slot,
  `$20-$23`; the other 28 banks, **224 KB**, would be free.
* **Every emulator the project uses accepts it, but one setup loses the
  saves**: snes9x, bsnes, ares, Mesen2, mesen-s and Mednafen's `snes_faust`
  give 256 KB. **Mednafen's default `snes` module masks the byte with 7, so
  `$08` reads as 1 KB, which it takes for "no SRAM"**: run as `mednafen
  abl.sfc`, without `-force_module snes_faust`, the game would not save at
  all. snes9x's libretro core hands its frontend only the first 128 KB to
  write to disk, which holds the slots.
* **The FX Pak does not write the card on every change.** It checksums the
  SRAM and, when it changed, writes the whole `.srm` —256 KB—. Without a
  `.msu`, in the background: a full pass every ~1 s with 256 KB, and the
  write once the SRAM has stayed the same for five passes. **With a `.msu`,
  which is how ABL ships, on every MSU-1 data seek and at most once a second
  otherwise, before serving the seek, and only while no MSU-1 audio
  plays.** The firmware update taro remembered is 1.9.0's "partial SRAM
  change detection", which only knows a table of commercial SuperFX and
  SA-1 games by the CRC of their ROM; 1.11.1 added `autosave` and
  `msu1_autosave` switches to `config.yml`, which turn it off altogether.
* **The DMA path works and is fast**: WRAM to SRAM through `$2180` and back,
  96 KB in **597-599 lines each way, 8.3 master cycles a byte, ~43 KB a
  frame**, the same in snes9x, bsnes, mesen-s and the audit core.
* **What it would buy, measured on ABL's map 52**: of the 80 black frames
  of the return from a battle (111 with MSU-1), the last **~25 are the map
  being rebuilt** —the grid decompressed again, the tilemap refilled, the
  tiles decompressed into VRAM again— and a stash would make them **~2-3**.
  The other 55 (86) are the audio package going back to the SPC700, which
  the SRAM cannot touch. It frees no WRAM by itself: bank 0 and `FARBSS` are where
  the walls are, and they hold what is read every frame.
* **What only the console can say** is how long the FX Pak keeps a MSU-1
  seek waiting while it checksums 256 KB and writes them. The test ROM
  shows it on screen; it is in `/shared/builds/sram-test/` with what to
  look at.

## What 256 KB is in ExHiROM

The header byte is a power of two in KB: `$05` is 32 KB, what the game
declares today (`runtime/header.s` in the-advocate), and `$08` is 256 KB.
In ExHiROM the cartridge decodes the SRAM at offsets `$6000-$7FFF` of the
banks whose address has bit 22 clear and bit 21 set —`$20-$3F`, and the same
with bit 23 set, `$A0-$BF`—, and takes the offset inside the SRAM from bits
16-20 and 0-12: `$20:6000` is byte 0, `$21:6000` is byte `$2000`, up to
`$3F:7FFF`, byte `$3FFFF`. Five bits of bank and thirteen of offset are 18
bits: **256 KB is the most that window reaches**, on the FX Pak
(`verilog/sd2snes_base/address.v`, mapper `010`, `IS_SAVERAM` and
`SAVERAM_MASK`) and in every emulator below. There is no overlap with the
ROM: in ExHiROM the ROM of `$00-$3F` and `$80-$BF` is only their upper
halves.

What is around it, from the emulators' sources:

* **bsnes** (the standalone, `EXHIROM-RAM` in `boards.bml`) and **mesen-s**
  also map the SRAM at `$70-$7D:0000-7FFF`, LoROM's place, and **Mesen2**
  maps it at `$80-$BF:6000-7FFF` always and at `$20-$3F` only without "strict
  board mappings" (off by default). Neither matters: the game already
  leaves the low halves of `$70-$7D` alone (`rm2k-snes/cartridge/src/banks.rs`
  in the-advocate) and reaches the SRAM only through `$20-$3F`. With
  Mesen2's strict mappings on, the SRAM is not there today either.
* bsnes-libretro, ares and Mednafen's `snes_faust` map `$20-$3F` and
  `$A0-$BF` only, the same as the FX Pak.

`$FFD8 = $08` does not move how anyone finds the header: the FX Pak and
Mesen score a point for a byte under 8, and the game carries its header
twice —at `$00FFC0` and `$40FFC0` of the file— with the same byte, so both
copies lose the same point. Measured: ABL's `release` with the byte
patched to `$08` boots to the same screens, at frames 120, 300 and 600, as
without it, in snes9x, bsnes and mesen-s.

## What each one does with it

| | Takes `$08`? | Largest | How it persists it |
|---|---|---|---|
| **FX Pak** (sd2snes 1.11.2, `smc.c`, `memory.c`) | yes, `1024 << n` | no cap below what the window reaches | checksums it and writes the whole `.srm` to the card: below |
| **snes9x** (`memmap.cpp`) | yes | `$09`, 512 KB | the standalone saves up to 256 KB for a HiROM; **its libretro core reports at most 128 KB** of `SAVE_RAM` (`retro_get_memory_size` in `libretro/libretro.cpp`), so RetroArch writes only that much |
| **bsnes** (`heuristics/super-famicom.cpp`) | yes | `$08` | the libretro core writes its own `.srm` when the game is unloaded, all of it; it exposes no `SAVE_RAM` |
| **ares** (`mia/medium/super-famicom.cpp`) | yes | `$08` —`$09` wraps to 2 KB— | its own file |
| **Mesen2**, **mesen-s** (`BaseCartridge.cpp`) | yes | `$08` | the `.srm`, all of it, on power off and on loading another ROM |
| **Mednafen**, `snes_faust` (`cart.cpp`) | yes | `$09` | on exit |
| **Mednafen**, `snes`, the default (`snes/src/cartridge/header.cpp:46`) | **no**: `1024 << (n & 7)` is 1 KB, read as "no RAM" | `$07` | — |

The last line is the only one that breaks something: the default module is
also the one without MSU-1, and [the emulators' page](../reference/snes/emulators.md)
already sends to `snes_faust` for that. With 256 KB it would also be the only
one in which the game cannot save.

## How the FX Pak saves

Read in the firmware's source (`src/snes.c`, `src/msu1.c`, `src/memory.c`,
`src/main.c`), 1.11.2. **It never writes a part: it writes the whole
`.srm`, the size the header declares.** When it writes depends on whether
there is a `.msu` next to the ROM:

* **Without MSU-1**, `snes_main_loop` runs every 250 ms and checksums 64 KB
  of the SRAM each time, so with 256 KB a full pass takes four calls, about
  a second. When a pass differs from the one before, it waits until **five
  passes in a row** come out the same, and then writes. If the SRAM keeps
  changing it writes anyway every ~50 changing passes (the "periodic save"),
  and it also writes on a reset. Writing is done by the cartridge's CPU
  while the console runs from its own memory: **the console does not stall**.
* **With MSU-1**, `msu1_loop` takes over, and there the check is
  `msu_savecheck`: the whole SRAM checksummed at once and written at once if
  it differs —no waiting for it to settle—. It is called **at most once a
  second** while idle, and **with no limit at the start of every data seek**
  (`prepare_data` calls `msu_savecheck(1)` before seeking), and in both
  cases **only while neither MSU-1 audio nor data is busy**
  (`is_msu_free_to_save`). While a MSU-1 track plays it neither checks nor
  writes; it writes on a reset.

So in ABL, which reads its battle backdrops from the `.msu`, a seek that
finds the SRAM changed and no MSU-1 track playing waits for a checksum of
the whole SRAM and for the card to take it whole. That happens today with
32 KB after saving the game; with 256 KB it costs eight times as much **even
when nothing in the SRAM changed**, because the checksum always reads all
of it. How much that is, in milliseconds, depends on the cartridge's CPU
and on the card, and is what the test ROM measures on the console.

**The firmware updates.** 1.9.0 brought "partial SRAM change detection for
known games (SuperFX/SA-1)": after loading, the firmware CRCs the ROM and
looks it up in `SramOffsetTable` (`snes.c`); a known game has only its save
area checksummed, so its work RAM does not keep triggering writes. A
homebrew is not in the table, and the table only narrows what is
checksummed: the write is still the whole `.srm`. 1.11.1 added the
`autosave` and `msu1_autosave` switches to `config.yml`: off, the FX Pak
writes only on a reset. Vendor and forum pages say the same
([stoneagegamer](https://stoneagegamer.com/fxpak-sd2snes-pro-classic-all-region.html?revpage=2),
[MiSTer forum](https://mail.misterfpga.org/viewtopic.php?p=42067)); the
source is what was read.

## The test ROM

`just sram-dma`, in `abl/` of the ports (`experiments/sram-dma/`): an 8 MB
ExHiROM cartridge like the game's, with `$FFD8 = $08`, a `.msu` and a
`-1.pcm`, so that the FX Pak runs it in its MSU-1 loop. A **round** —A for
one, B for one after another— fills `$7E:8000-$7F:FFFF` (96 KB: what a
battle takes from the map) with a different 96 KB of ROM each time, copies
it to SRAM banks `$24-$2F` by DMA through `$2180`, B bus to A bus, in twelve
8 KB pieces —an A-bus address does not carry into the next bank, and the
window is 8 KB a bank—, wipes the WRAM, brings it back and checks its sum.
With MSU-1 it times a data seek right after the stash, and another one
after that, with the SRAM as the FX Pak last saw it. At boot it measures
how much SRAM there is, by writing each bank's number at its `$7FF0` from
the top down and looking for the first that a lower one overwrote, and
says whether the last session's stash survived turning the console off.
Banks `$20-$23` are never written but for a mark, which is checked after
every round.

In the harness (`measure.py`), the same in snes9x, bsnes, mesen-s and the
audit core, within two lines:

| | 96 KB in | master cycles a byte | bytes a frame |
|---|---|---|---|
| WRAM to SRAM | 597-599 lines | 8.3 | ~43,000 |
| SRAM to WRAM | 598 lines | 8.3 | ~43,000 |

8 master cycles a byte is the DMA's own speed, and the rest is the 40
cycles a line of DRAM refresh. All four saw 256 KB, kept the slots' mark,
and found the stash intact after a reset. The emulators seek the MSU-1 at
once, so the seek lines read zero there.

## What a stash would save, measured on ABL

The case `battle-in-52` fights on map 52, the one with the biggest grid
(20,000 B, and 4,850 B of blocks and bytecode behind it). Its return from
the battle, frame by frame in snes9x, comparing WRAM and VRAM with what the
converter wrote for the map (a copy of `release` built earlier the same
day: the installed tools were behind the runtime, and nothing on this path
had changed):

| Frames | What |
|---|---|
| 55 (86 with MSU-1) | the map's audio package back to the SPC700 |
| ~2 | sprites, the window, the palettes |
| ~6 | `lz2_grid`: the grid and what lives behind it, ~25 KB |
| ~6 | `fill_window`: the tilemap, written from the grid |
| ~12 | `back_to_map`'s `load_map` and `load_above`: the tiles, decompressed through `LZ2_BUFFER` into VRAM |

The last ~25 frames are rebuilding what was there before the battle. With
the map **stashed when the battle starts** —the part of `$7E:8000-$7F:FFFF`
the map uses, and the VRAM's tiles and tilemaps, which an SRAM-to-`$2118`
DMA puts back like any A-bus to B-bus one— they become the DMA back: ~0.6
frame for the 25 KB of WRAM and ~1-1.5 for the VRAM, and the same again on
the way in, inside the black that already covers the battle's own package.
**About 22 frames, 0.37 s, of the 80 the black lasts on map 52 (111 with
MSU-1).** On a
median map —1.7 KB of grid and what lives behind it, 7 KB of tiles— the
same arithmetic gives a few frames. A door's load across all 313 maps
(median 8 frames, worst 95) is dominated by each map's audio package, so the
same cache would shorten it much less than its share of bytes: measured
against the bytes decompressed, the correlation is 0.52.

The SPC700's 55-86 frames are out of reach: it is fed through its four ports,
and the SRAM is one more place on the A bus, no closer to it than the ROM.

## Proposal

**The layout**, with 256 KB declared:

| Banks | Size | What |
|---|---|---|
| `$20-$23` | 32 KB | the four save slots, as today; nothing else touches them |
| `$24-$2F` | 96 KB | **the map across a battle**: `$7E:8000-$7F:FFFF` as it is, or only what the map uses —the grid, what lives behind it, the ChipSet animation's drawings— |
| `$30-$37` | 64 KB | **the map's VRAM across a battle**: its tiles, the upper layer's and the tilemaps, taken by DMA from `$2139` before the battle draws, put back to `$2118` after |
| `$38-$3F` | 64 KB | free: a cache of the last map's decompressed streams, for a door that goes back, or the teleport's carried bytecode (`CODE_CARRY`, today 19,968 B in `$7F` with a cap) |

A stash carries a header with a nonce drawn at boot, and is trusted only
within the boot that wrote it: the FX Pak will write it to the card and
it will be there at the next power on.

**The order of steps, entering a battle**, for the FX Pak: the backdrop's
seek in the `.msu` **before** the stash, so the seek does not find a
changed SRAM to write first. Leaving, the stash is read, which changes
nothing.

**What it costs**:

* On the FX Pak, one write of the whole 256 KB `.srm` **per battle**, at the
  next seek or second with no MSU-1 track playing; and eight times the
  checksum of today on every seek, changed or not. The test ROM says how
  long each is; if the seek after a stash is long, the stash has to go after
  the seeks, or the game has to live with it.
* On Mednafen's default module, no saves. Either the documentation says
  `-force_module snes_faust` always —it already does for MSU-1— or the
  game keeps `$05` and the idea waits.
* A path for the suite: the harness reads the SRAM through snes9x's
  `SAVE_RAM`, 128 KB, so a test that looks at a stash reads it through WRAM
  after it comes back, or through `$3F` with another core.
* SD wear: one 256 KB write a battle is nothing for a card.

**What it does not do**: free WRAM. What the battle tramples is rebuilt
today; the stash only makes it faster. The walls the
[WRAM and SRAM map](2026-10-15-the-wram-and-sram-map.md) found are bank 0's
and `FARBSS`, which hold what is read every frame and cannot live behind a
DMA.

## What was not measured

* **The FX Pak itself**: the checksum's and the write's time, whether the
  MSU-1 hum stutters, whether anything stalls. That is the test ROM's job,
  on the console.
* **SRAM to VRAM and VRAM to SRAM by DMA**: the same DMA unit, A bus to B
  bus, as the measured WRAM path, but no test ran it.
* **The stash in the game**: nothing of it is implemented; the frames above
  are what the return spends today and what the test ROM's DMA would
  spend instead.

## How it was measured

* **The sources**, read in the clones of each project at the commits of
  the header; the files are named in each section.
* **The test ROM**: `just sram-dma` in `abl/` of the ports, branch
  `aguara-sram-dma`: it builds it and runs `measure.py`, which presses A
  four times in each core, reads the lines from WRAM (`RESULTS` in
  `experiments/sram-dma/sram.s`), resets and reads what the boot found.
* **The header**: ABL's `release` with `$FFD8` patched to `$08` and its
  checksum fixed, booted beside the unpatched one in snes9x, bsnes and
  mesen-s, compared at 15 bits at frames 120, 300 and 600.
* **The return from battle**: the case `battle-in-52` played as
  `test_the_grid_comes_back_whole_from_a_battle` plays it (A every nine
  frames until the battle closes), and from then on, every hardware frame:
  whether `PKG_DEST`/`PKG_S` moved, how much of the grid and of what lives
  behind it equals the converter's `raw/` streams, how many bytes of VRAM
  and of `$7E:8000-$7F:FFFF` changed since the frame before, and whether the
  screen is black. Once with MSU-1 and once without.
* **The doors**: every map of `release` loaded with `_load` of
  `test_decompressed_load.py`, against the sum of its `raw/` streams.
