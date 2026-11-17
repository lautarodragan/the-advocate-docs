# Far RAM in the SRAM

> **Read and measured on 2026-11-17** by Aguará, with the-advocate at
> `269bb12c`, the ports at `a311082` plus the test ROM of branch
> `aguara-sram-far` (`a3861bf`), and ABL's `release` built from them. The
> FX Pak's behaviour was read in sd2snes `cf7e21d` (firmware 1.11.2): its
> `src/msu1.c`, `src/memory.c`, `src/lpc175x/sdnative.c`, `src/crc.S`,
> `config-mk3` and `verilog/sd2snes_base/main.v`. It is a snapshot.

A follow-up to [The SRAM as slow WRAM](2026-11-16-the-sram-as-slow-wram.md),
which said the SRAM "frees no WRAM". taro and Hornero, rereading it: the
CPU reaches the SRAM directly and as fast as WRAM, so far-RAM variables
could live there for good —`FARBSS` is what keeps `baked_events` off in
ABL—. taro's worry, in the task
[`the-sram-as-slow-wram`](https://github.com/lautarodragan/the-advocate-work/blob/main/tasks/the-sram-as-slow-wram.md),
is the FX Pak with the `.msu`: whether music plus the periodic checksum
can add lag. It is a new page and not a section of the old one because it
answers another question, at other commits, and a research is not
updated.

## Summary

* **The speed is the same**: the SRAM window, `$20-$3F:6000-7FFF`, is 8
  master cycles a byte, like WRAM. What changes is everything around it.
* **About 4 KB of `FARBSS` is reached only with long addresses** and could
  move with its segment's name changed and a clear at boot; the rest,
  20 KB, is reached through 16-bit addresses with the data bank at `$7E`
  or through `$2180`, and each needs code. The ~700 B `baked_events`
  needs (`posed.s`) is one of the easy ones: one line to change.
* **The FX Pak never stalls the console**, but **with a `.msu` it
  checksums the whole declared SRAM before every MSU-1 data seek made with
  no track playing, and writes the whole `.srm` if it changed**. Variables
  in the SRAM make it changed every time. Estimated from the firmware: a
  seek that today waits ~25-35 ms of checksum would wait **~0.5-0.7 s with
  256 KB** declared, and ~0.13-0.2 s with 64 KB —ABL gives up a seek
  after ~0.35 s and loses the sound effects of that package or the
  battle's backdrop—. In ABL those seeks happen on a map change with a new
  song and on entering a battle, with the track stopped.
* **taro's reading is right, with two corrections**: playing a track, it
  neither checks nor writes (true); without one it checks **at the next
  data seek**, and the once-a-second check only runs while the data side
  is idle, which in ABL is almost never after the first seek; and every
  check that finds a change writes all of it, so with variables there it
  is not "a CRC at each seek" but a CRC **and a 256 KB write** at each.
* **The recommendation is no**, for `baked_events`: the 700 B are cheaper
  behind the grid, in `$7F`, where Mulita is moving them. The SRAM is
  worth it only for what changes once per battle, not every frame. The
  test ROM's new frame test, in `/shared/builds/sram-test/`, is what turns
  the estimates into numbers.

## 1. What could move, and what cannot

### The speed

`$00-$3F:6000-7FFF` and `$80-$BF:6000-7FFF` are 8 master cycles a byte,
the same as WRAM in any bank and as SlowROM: fullsnes's "SNES Memory Speed"
table ("SRAM and Expansion", Medium 2.68 MHz, 8 clocks), and bsnes, which
gives 8 clocks to any address with `address + $6000 & $4000` set
(`CPU::read` in `sfc/cpu/memory.cpp`). A long `lda f:` to `$24:6000`
costs what one to `$7E:6000` does. The FX Pak serves those cycles from
its own memory, without waiting states.

### Who is in `FARBSS`, and how they are reached

`FARBSS` is `$7E2000-$7E7FFF`, 24 KB with the switch table; in ABL's
`release` it is full to the byte —`FARBSS` 24,436 B and `SWITCHES` 128 B,
12 B left (`just space`)—. Each of its labels was classified by how the
runtime reaches it: a script read the labels and their sizes from
`abl.sfc.dbg` and every line of `runtime/` that names them or a constant
derived from them. It is a heuristic over text —a label's size is the
distance to the next one, and an access through an offset of another
label counts for that one—, good for the order of magnitude:

| How it is reached | Labels | Bytes | What moving it takes |
|---|---|---|---|
| **only long** (`f:`, a far label, `^label` for the bank) | 206 | ~4,200 | its `.res` in another segment, and the boot clearing it |
| **16-bit** (`.loword`, with the data bank at `$7E`, or `f:FAR_LENGTH + X`, which writes the bank by hand) | 125 | ~8,200 | each routine that sets the data bank to `$7E`, or each `FAR_LENGTH` |
| **through `$2180`** too (WMADD, the save's DMA) | 13 | ~12,100 | a different copy: `$2180` reaches WRAM only |

The biggest of the easy ones: the music's state (`FAR_MUSIC`, 1,024 B),
the audio package's bookkeeping (`FAR_ARAM_OF` 384, `INT_INIT` and
`INT_END` 256 each, `FAR_WANTS` 192), `applied_page` (240), `PANO_ON`
(234), `CACHE_N` (200). The biggest of the hard ones: the text layer's
buffer (`far_text_layer`, 8,810 B, 700 lines with the data bank at
`$7E`), the battle (`far_battle`, 1,600 such lines), the animation's
buffer (`ANIMB_BUF`, 2 KB).

**`posed.s`, what `baked_events` needs** (~700 B: `POSED_REG`,
`POSED_LAST`, `POSED_T`), is all long except `posed_queue`, which sets the
data bank to `^POSED_REG` —so it follows the label— and reads one other
tenant, `FAR_VRAM_MAP`, with a 16-bit address: one line to make `f:`. Its
DMA to VRAM takes the bank from the label too (`lda #^POSED_REG`), and
SRAM is on the A bus like WRAM.

A data bank of `$20-$3F` is, if anything, better than `$7E`: absolute
addresses reach the low 8 KB of WRAM (bank 0's `BSS`), the hardware
registers and the SRAM window, all three. What breaks is a routine that
also reads another far tenant that stayed in `$7E`.

### What cannot move

* **Bank 0, the zero page and the stack**: the direct page and the stack
  live in bank 0, and in ExHiROM there is no SRAM in `$00-$1F`. The
  `BSS` is read with absolute addresses everywhere.
* **What goes through `$2180`**: WMDATA is a port to WRAM only. The save
  (`save_table`, a DMA from `$2180` to the SRAM) would become an
  SRAM-to-SRAM copy, which the DMA cannot do: an `mvn` instead, 7 cycles a
  byte.
* **What a DMA writes to WRAM through `$2180`**, such as the LZ2
  decompression's output.
* **An object larger than 8 KB, or indexed across `$7FFF`**: each bank's
  window is 8 KB, and `$24:7FFF + 1` is `$24:8000`, ROM, not `$25:6000`.
  The linker needs one memory area per window, so nothing straddles two;
  `far_text_layer`, 8,810 B, does not fit in any.

### What else it asks

* **The SRAM keeps the last session**: in every emulator and on the FX
  Pak it comes back from the `.srm`, so what moves there has to be cleared
  at boot like WRAM —and the boot must not touch banks `$20-$23`—.
  `test_cores.py`, which catches what depends on uninitialised WRAM by
  comparing bsnes's garbage with snes9x's zeros, sees nothing here: both
  load the same `.srm`.
* **The declared size**: with 256 KB (`$08`), Mednafen's default `snes`
  module sees no SRAM (the old page), and then the game would not only
  not save, it would not run. 64 KB (`$06`) or 128 KB (`$07`) work there,
  and 64 KB is enough for 32 KB of far RAM beside the slots.

## 2. The FX Pak with an SRAM that changes every frame

Read in the firmware, 1.11.2. ABL runs in `msu1_loop`, because its builds
carry a `.msu`.

**The console is never held.** The MCU reads the SRAM through the FPGA,
which serves an MCU request only in a slot the console is not using
(`ST_IDLE`, `free_slot`, in `main.v`): the console keeps its 8 cycles, the
MCU waits. What waits is whatever the console asked of the MSU-1 while the
MCU is busy: the MCU runs one loop, and while it checksums or writes it
does not serve a seek or a track start, whose busy bit stays up.

**When it checks**, `msu_savecheck` (checksum of `sramsize_bytes`, all
that is declared; `save_srm` of all of it if the checksum changed):

* at the start of **every data seek** (`prepare_data`), immediately, if
  no MSU-1 track plays and the data side is idle —which `prepare_data`
  sets itself just before asking—. The seek's busy bit stays up
  meanwhile;
* **at most once a second in the loop**, if both sides are idle. After a
  seek the data side stays busy until a read reaches the end of the
  `.msu`, so after a game's first seek this check stops, and checking
  happens only at the next seek. In ABL, whose seeks read 16 KB from
  places in the middle of a `.msu` of megabytes, that is almost always;
* when a track is chosen (`prepare_audio_track`), at most once a second,
  if the data side is idle (so, again, rarely in ABL);
* on a reset (`save_during_msu_shortreset`), always. **Not on power off.**

While a track plays (`msu_audio_usage` busy from "play" until it is
stopped or a non-repeating track ends), none of these run.

### taro's reading

> If I understand correctly, precisely playing music would make it
> persist and check _less_, not _more_, and when no music is playing
> (practically never), it would be at most once per second, but it would
> do a 256k write per second...

Right about the music: playing a track, nothing. Without a track, two
corrections. The once-a-second check needs the data side idle, which in
ABL it is not after the first seek; so it is **once per seek**. And each
check that finds the SRAM changed writes it whole: with variables there,
**every seek with no track playing waits for a checksum and a 256 KB
write**. And "practically never" is not ABL's case where it matters:
`map_song_mute` (`enter_map.s`) stops the track before the new map's
audio package, whose sound effects come from the `.msu` (one seek a
package, `se_msu_upload`), and a battle's backdrop is a seek too.

### How long, estimated

Nothing here is measured: the MCU's loops were counted in the source, and
the frame test below is what measures them.

* **The checksum**, `calc_sram_crc`: a byte at a time over SPI at 48 MHz
  (`SSP_CLK_DIVISOR = 2` of a 96 MHz CPU), with `FPGA_WAIT_RDY` (sixteen
  `nop`s and two polls), a reset check and a table CRC-32 per byte: about
  70-100 cycles, **0.7-1 µs a byte**. 32 KB, what ABL declares today:
  **~25-35 ms**; 64 KB ~50-65 ms; **256 KB ~0.2-0.27 s**.
* **The write**, `save_sram`: the SRAM read again the same way in 512-byte
  pieces, each written by bit-banging the card's four data lines
  (`send_datablock`: two nibbles, two clocks and the four lines' CRC-16 a
  byte, ~0.45 µs), plus the card's own busy time per block and the FAT's
  (the file is re-created, `FA_CREATE_ALWAYS`): **~1.2-1.5 µs a byte plus
  the card**, so ~0.3-0.4 s for 256 KB, ~0.1 s for 64 KB, before what the
  card adds —from milliseconds to over 100 ms on a slow card—.

| Declared | A seek, no track, SRAM unchanged | A seek, no track, SRAM changed |
|---|---|---|
| 32 KB (today) | +25-35 ms | +70-100 ms |
| 64 KB | +50-65 ms | +0.13-0.2 s |
| 256 KB | +0.2-0.27 s | **+0.5-0.7 s** |

With a track playing, each is only the seek, a 16 KB read of the card. ABL's
`msu_data_seek` gives up after `$FFFF` passes of 114 master cycles,
**~0.35 s**: with 256 KB and a changed SRAM, the backdrop would fall back
to the ROM's and a package's sound effects would not play. With 32 KB
today the cost already exists —a checksum on each such seek, and a write
after the game was saved—, small enough not to have been noticed.

### Wear, and the saves

Each write is the whole file. With variables in the SRAM and no track
playing, 256 KB per seek; in the frame test's WRITES mode, 256 KB a
second, ~0.9 GB an hour of silence. And the `.srm` reaches the card only
at a check or a reset: a game saved during a track and the console turned
off before the next silent seek loses that save —today too, with or
without far RAM in the SRAM—.

## 3. The test ROM's frame test

`just sram-dma` in `abl/` of the ports now builds two cartridges, the
256 KB one and a 64 KB one (`--sram-kb 64`, without rounds: its stash
would land on the slots' banks), and the X button steps a **frame test**:

| Mode | Every frame | Every 30 frames |
|---|---|---|
| OFF | — | — |
| WRITES | 16 bytes of `$24:6000` | — (entering it seeks the `.msu`'s last sector, which reads to its end and leaves the data side idle, so the once-a-second check runs: the LED should blink) |
| SEEKS | — | a timed data seek |
| SEEKS+WRITES | 16 bytes | a timed data seek |

It shows the seeks' last, mean and longest milliseconds and the **late
frames**, those in which the main loop did not run: the lag a player sees.
In snes9x, bsnes, mesen-s and the audit core, both cartridges: their SRAM
seen whole, 10 seeks in 300 frames, **0 late frames**, the bytes in bank
`$24` following the frame. The emulators answer a seek at once, so the
timing path runs but never times anything: **the numbers are the FX
Pak's**, in `/shared/builds/sram-test/README.txt` the eight to send —two
cartridges, music on and off, SEEKS and SEEKS+WRITES—. SEEKS+WRITES minus
SEEKS is the write; SEEKS without music minus with it is the checksum; and
64 against 256 KB, how both scale.

## 4. For `baked_events`: the SRAM, or behind the grid

What `baked_events` needs in ABL is `posed.s`'s ~700 B of far RAM, and
`FARBSS` has 12. The two ways:

| | Far RAM in the SRAM | Behind the grid, in `$7F` ([Mulita's task](https://github.com/lautarodragan/the-advocate-work/blob/main/tasks/event-page-records-compressed-behind-the-grid.md)) |
|---|---|---|
| Code | `posed.s`'s block to a new segment, one line to `f:`, the boot clearing it, the header's byte, a memory area per window | `POSED_REG` to the room behind the grid, and rebuilt after a battle, which tramples `$7F` |
| On the FX Pak | every seek with no track pays a checksum and a write of the whole SRAM: estimated +0.13-0.2 s at 64 KB, +0.5-0.7 s at 256 KB, past ABL's seek cap | nothing |
| Emulators | 256 KB breaks Mednafen's default module; 64 KB does not | nothing |
| What tests see | uninitialised SRAM is invisible to `test_cores.py` | the same as any WRAM |
| Room | 32 KB at 64 KB declared, more at 256 | what each map leaves behind its grid; map 52, the fullest, has its grid's 20 KB and 4.85 KB more |

**Recommendation: behind the grid.** The SRAM is fast for the console and
costly for the FX Pak exactly when it changes, and far RAM changes every
frame: the music's state, the events' pages, the poses. It would turn the
silent seeks of map changes and battles —cheap today— into half a second,
on the cartridge taro plays. What the SRAM suits is what changes once in a
while and is big: the map's stash across a battle, one write per
battle, with the backdrop's seek before it. If the FX Pak's numbers come
out far below these estimates, the table's middle row changes and this
with it; a 64 KB SRAM would then be the one to try, for Mednafen's sake.

## How it was measured

* **The tenants**: `abl.sfc.dbg` of ABL's `release` (built 2026-11-17
  00:08 in the ports' main checkout, at the commits of the header), its
  `FARBSS` and `SWITCHES` labels with the distance to the next as size,
  and every line of `runtime/` that names one or a constant derived from
  one, classified by the text of the line; `posed.s` read by hand.
* **The firmware**: read, and its per-byte loops counted by instruction
  for the estimates; the clocks from `config-mk3`.
* **The frame test**: `just sram-dma`, both cartridges in the four cores.
  Not run on the FX Pak.
