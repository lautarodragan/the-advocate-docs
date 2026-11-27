# The `.msu` file: data the MSU-1 serves to the ROM

The MSU-1 has two parts: the audio tracks (`<rom>-N.pcm`, which
`advocate build` renders, see `rm2k-snes/rm2k-snes-audio/src/tracks.rs`) and a
**data port** that reads `<rom>.msu` as a stream. Until 2026-09-25 the
`.msu` was empty and only served to turn the MSU-1 on. Since that day it
carries **the battle backdrops**, which started as a pilot, and since
2026-09-27 **all of the game's**, whether or not they are in the
cartridge's maps: see "The backdrops" below. And since 2026-09-26, behind
them, **the quality sound effects** of each audio package: see "The
effects" below. And since 2026-10-11, only in a game whose ROM does not
reach, behind everything, **what does not fit in the ROM**: see "What does
not fit in the ROM" below. And since 2026-11-16, in the same region and in
every game that has it, **the debug screen's font, names and windows**: see
"The debug screen" below.

The ROM carries the backdrops the cartridge's battles ask for, and **light
ones**: 128x64 at 4bpp and 16 colors, which the ROM doubles in size,
against the 256x128 at 8bpp and 96 colors of the `.msu` —see `scene.rs` in
`rm2k-snes-convert`—. That way you can tell by eye which side the backdrop
came from, and each one costs in the ROM an eighth of what it weighs in the
`.msu`. A battle whose backdrop the ROM doesn't carry uses, without MSU-1,
a fallback one.

## The rule

**Without MSU-1 the game works the same.** taro plays in Mednafen, which
doesn't have it (in its default module; see [the
emulators](../reference/snes/emulators.md)). So what comes from the `.msu`
**replaces** something the ROM already has, never adds it: without MSU-1,
with a `.msu` from another build, empty or truncated, or if the MSU-1
doesn't finish seeking in time, the ROM uses its own.

**The one exception is a game whose ROM does not reach**, and it is the
project's decision, not the converter's: see [the MSU-1 is required only
when the ROM does not reach](../decisions/the-msu-1-is-required-only-when-the-rom-does-not-reach.md)
and "What does not fit in the ROM" below. **And the other is the debug
screen**, which is not the game: without the MSU-1 the pause menu does not
offer it, and the rest is the same game. See "The debug screen" below.

## The port, from the console's side

```text
$2000..$2003  write: the position, 32 bits little-endian. The write
              to $2003 is the one that seeks.
$2000         read: bit 7 set while seeking ("data busy")
$2001         read: the next byte; advances by itself
```

It is read by DMA with the source **fixed** at `$00:2001` —bit 3 of
`DMAP`—, which runs at the speed of any DMA, 8 master cycles per byte:
32 KB are about three quarters of a frame, the same as from ROM. Which
emulators implement it: in [the emulators](../reference/snes/emulators.md).

## The format

`rm2k-snes-audio` builds it, in `msu_data.rs`, and its documentation is the
reference: an 8-byte signature (from the sha256 of what follows), `"ABLD"`,
the version, an index of `(position, length)` per entry, and the entries.
Each backdrop is two: its 512 tiles of 8bpp and its 96 colors, exactly as
they go to WRAM and CGRAM. The ROM has something else in `backdrop_dir` and
`backdrop_cgram`: the light fallback, which `scn_fill` and `scn_double`
expand to 8bpp and double in size.

The layout, since version 2 (2026-09-27):

```text
$000000  the header and the index, padded with zeros up to $10000
$010000  the game's backdrops, by number: backdrop k takes $8200
         bytes from $10000 + k × $8200 —its tiles, and $8000
         further on its colors; the rest is padding up to the
         512-byte sector—
behind   the effects, from the end of the last backdrop place
```

**The backdrop region is fixed**: backdrop k is at the same byte in the
`.msu` of any cartridge, because a backdrop's number comes from the game
and not from the maps that are carried (see "The backdrops"). In version 1
they went right behind the index, and the index grows with each
cartridge's effects: a backdrop's place moved with any audio change. How
many backdrops there are and how much the `.msu` weighs is printed by the
converter, in `converter.log` ("`.msu:` N fondo(s) y M efecto(s) en B
bytes", that is, N backdrop(s) and M effect(s) in B bytes).

The converter writes it —it is the one that has the bytes and decides the
order—, next to each `.sfc` and with its name: `advocate build` passes
it `--msu`. Both cartridges carry it, and the test of the MSU-1 path
runs on `release`, which is the ROM that is played.

## How the ROM knows the `.msu` is its own

The converter emits in the `.inc` the signature it wrote
(`msu_data_signature`) and where each backdrop ended up
(`msu_backdrop_pos`, by backdrop number: it is the arithmetic above, in a
table so the ROM doesn't multiply). When a battle with a backdrop starts,
`msu_data_valid` —in `runtime/msu1.s`— seeks position 0 and compares
the 8 bytes; if they match, `scn_fill` and the colors of `scene_upload`
—in `runtime/battle_scene.s`— read from the `.msu`, and `SCN_MSU` is
left at 1.

**The signature is of the content, not of the commit**: a `.msu` from
another build with the same backdrops is accepted, and that is fine,
because they are the same bytes in the same positions. (The light
fallback, "The light fallback", didn't change the `.msu`: the quality one
is the one from before, byte for byte, and with it the signature.) What it
does **not** cover: a `.msu` with the header intact and the data damaged.
The ROM can't hash 32 KB per backdrop at the start of each battle.

The positions come from the `.inc` and not from the file's index: once the
signature is accepted they are the same data, and reading the index would
be one more seek per battle. The index stays for the tools
(`rm2k-snes-audio msu-index`) and for adding entries without changing the
format.

## The backdrops

**The game's backdrops** are all the ones a battle can ask for: the ones
named by the `EnemyEncounter` and `ChangeBattleBG` commands of the whole
game, those of the map tree and those of the terrains. They are sorted by
name ignoring case, which is how RM2k looks them up, and each one's place in
that list is its **number**: `scene.game_backdrops`. A name that is in
neither the game nor the RTP (`RTP_DIR`) doesn't go in; that battle
goes without a backdrop, as in RM2k.

Which backdrop a battle asks for is RM2k's rule —`Game_Map::SetupBattle`
and `Spriteset_Battle` in easyrpg-player—: the one the `EnemyEncounter`
names; or, if it is a random encounter or the command asks for "the one of
the place", the map's, going up the tree; and if the map names none, **the
one of the terrain of the cell the hero stands on**. The ROM resolves that
one when the battle starts (`scn_terrain`, in
`runtime/battle_scene.s`), with `backdrop_terrain`. That is how the
battles in the world come out.

**The ROM carries all the ones the cartridge's battles can ask for**: the
ones the troops name, those of the maps with random encounters and those of
the terrains of those maps when they name none —see `scene.rs` in
`rm2k-snes-convert`—. Without MSU-1, a battle whose backdrop the ROM
doesn't carry uses **the fallback**, `[battle] fallback_backdrop` in the
project file —why that one, in its comment—: `backdrop_fallback` says, by
backdrop number, which of the ROM's is drawn. taro asked for it on
2026-09-27, because the battles in the world came out black, and black is
confusing; the same day they asked for the ROM to carry all of `release`'s,
and since then the fallback is left for a troop that fights "of the place"
in a map without encounters, where the converter can't know what terrain
the hero will be standing on. With MSU-1 each battle reads its own from the
`.msu`. The ROM carries two numbers per battle: `SCN_BACKDROP`, the game's,
which is the one looked up in the `.msu`, and `SCN_FALLBACK`, the ROM's.

What each backdrop the ROM carries costs is printed by the converter
("fondo de batalla … en la ROM … B"): 4,096 bytes of tiles and 32 of
colors, which is a property of the format, plus 4 of pointer.

## How long it takes

It is measured with `SCN_FRAMES`, which `scene_upload` leaves in RAM when
opening the battle: `tests/abl/test_msu_data.py` in the ABL port builds
both paths, and reading it there is one line. In snes9x and in bsnes the
`.msu` takes **less** than the ROM —measured in the droid battle—: both are
DMA at the same speed and seeking is instant there, but the fallback takes
one DMA per tile for the fixed planes and is then enlarged by the CPU,
pixel by doubled pixel: about four frames more than when it was 256x128
—measured with `SCN_FRAMES` in the world battle, on 2026-09-27—.

**Mednafen is the only emulator in which seeking takes time**
(`-force_module snes_faust`; what each one does, in [the
emulators](../reference/snes/emulators.md)): each seek costs one or two
frames, and there are four per battle —the signature, the colors and the
tiles for each of the two WRAM buffers—. Measured on 2026-09-26 in the
droid battle, played from the title screen with the map's track playing:
`SCN_FRAMES` at 11 with the `.msu`, against 5 without it in Mednafen and 4
with it in snes9x and bsnes. That is about seven frames of waiting, a little
over a tenth of a second. Mednafen's RAM is read by saving the state (F5):
the `.mc0` in `~/.mednafen/mcs/` is a gzip, and the WRAM goes whole in its
`WRAM` entry.

On the FX Pak seeking inside its 16 KB buffer is immediate, and outside it
means reading the SD card —`prepare_data` in `src/msu1.c` of sd2snes, its
firmware—. That is **not measured**. The cap on the wait —`MSU_WAIT`, in
`runtime/msu1.s`— is about ten times what Mednafen takes.

If a seek doesn't arrive in time, what was already read from the `.msu` is
the quality backdrop and what follows would come from the light one, and
they share neither colors nor tiles: `scene_upload` notices and redoes the
whole backdrop from the ROM. No emulator takes long enough to cut the wait,
so that path **is not run by any test**.

## After a save: a seek nobody reads

With a `.msu` the FX Pak writes the SRAM to its card only while no MSU-1
track plays: at a data seek, at a track change, once a second while idle,
or on a reset —`msu_savecheck` in its firmware's `src/msu1.c`—. Turning the
console off writes nothing, so a game saved while its song played from a
track was lost at power-off
([the task](https://github.com/lautarodragan/the-advocate-work/blob/main/tasks/a-save-on-the-fx-pak-waits-for-a-silent-seek.md)).
Since 2026-11-17 `save_game` ends in `msu_save_flush`, in `runtime/msu1.s`:
with a track playing it fades it out, pauses it, waits for `$2000` to say it
stopped, seeks to 0 and waits for the data busy bit —the firmware clears it
after writing the `.srm`—, plays the track again where it was and fades it
back in; with none playing, the seek alone. Nothing reads what that seek
points at, and nothing reads the port as a stream across a save: every
reader above seeks before it reads.

What it costs on the FX Pak was measured with the SRAM test ROM, not with
the game: with a track playing a seek takes about 2 ms, and with none about
1.5 ms per KB of the declared SRAM to checksum it and 2.6 ms per KB more to
write it when it changed. The routine's comment has the reasons for each
step; the order is checked on bsnes's MSU-1 log, by the save tests of
ABL's `test_msu.py`.

## The effects

Behind the backdrops go the sound effects of each audio package, whole and
each at the rate that package allows with ARAM free of music —the split
belongs to `rm2k-snes-audio`, in its `msu.rs` and `rates_per_package`—: by
package, in the order of the table, and inside each one in SRCN order,
which is how `upload_placed` uploads them. That way a package that goes up
at once —a battle's, the boot's— **seeks once** and reads straight
through; an effect that is in two packages goes twice. A teleport's go up
after its fade, in pieces in the idle end of the ticks or whole when one is
asked for, seeking again whenever another reader moved the stream: see
[a teleport's sound effects go up when they are
needed](../optimizations/lossless/audio.md#a-teleports-sound-effects-go-up-when-they-are-needed). `cartridge::msu` in `rm2k-snes-audio` writes it, and also emits
what the ROM needs: `se_msu_pkg` and `se_msu_rows` —where each effect of
each package is and how long it is— and `se_msu_pitch_pkg` and
`se_msu_pitch` —the pitch of each sound with each package's version, only
for the sounds whose effect comes from there, and two packages with the
same list share it—, and next to it a readable `effects.json`.

The signature is the same as the backdrops' and is checked once per
power-on, at the first package load (`se_msu_prepare`, in
`runtime/msu_effects.s`). Without the signature, the effects come
from the ROM as always. A seek that doesn't arrive in time leaves that
effect not uploaded: it doesn't sound, because the ROM's version has a
different pitch. `tests/abl/test_msu_effects.py` in the ABL port covers
it, except for that wait, for the same reason as the backdrops'.

## What does not fit in the ROM

A game whose ROM does not reach —Vampires Dawn, on 2026-10-11: 12.6 MB of
data for the 7.94 the cartridge has, and not enough even compressing the
poses— says so in its project file, `[rom] msu = "required"`, and then
what does not fit travels in the `.msu`. **The mechanism is the same for
every game**; what is particular to one is only that its ROM does not
reach. Without the setting nothing of this exists: the `.inc` and the ROM
are byte for byte the ones from before it existed.

**What may go** is marked by its emitter, with a `.msu` in front of its
`.block`, and only what the ROM reads through something that knows how to
read the port, and **when a map or a battle loads, never on every frame**:
a seek costs Mednafen one or two frames (see "How long it takes" above).

```text
the map's LZ2 streams      tiles, upper layer, sprites, the ChipSet
                           animation, the grid and the bytecode
the battle's LZ2 streams   the battlers and the light backdrops
a big picture's tilemap    LZ2
a panorama's tiles         LZ2, since 2026-11-16
                                  all four through `lz2_decompress`,
                                  which sends them to `lz2_msu`
a big picture's tiles      one DMA, with the screen off, through
                           `msu_dma0`
the samples                `upload_sample`, which uploads them to ARAM
                           like the `.msu`'s effects: `msu_sample_l`
```

**The poses stay in the ROM**, and they are the biggest thing there is
(4.3 MB in Vampires Dawn): they are uploaded on the fly, in the vblank —see
[poses are uploaded on the fly](../optimizations/lossless/characters-and-sprites/poses-on-the-fly.md)—,
and a seek per pose is not affordable. Nor what the CPU reads while the map
is played: the event pages, the panorama's tilemap, the music streams, the
texts.

**Which ones go is decided by the bank packing**, `pack_with_msu` in
`rm2k-snes-cartridge`, which is the one that knows what fits: the fewest
possible, from the largest to the smallest, until the rest fits. In
Vampires Dawn that is almost everything that may go —1,533 pieces, 5.3 MB—,
because what stays is 7.10 MB for the 7.94. If even with all of them out
it does not fit, it says so, and how many there were.

**The region** goes behind the entries the index describes, from the next
512-byte sector, and the packing writes it after the converter wrote the
rest:

```text
start          8 bytes: the region's signature, from the sha256 of
               everything from start + 256 on
start + 256    the pieces, each from a multiple of 256 —its "unit"—
               and padded with zeros to the next one
```

It is not in the index, and **the `.msu`'s signature covers up to where
the entries end, not the region**: that is how the region is appended
without the other signature changing —for a `.msu` without a region it is
the same signature as always—. Each piece that went leaves its label as a
constant: bank `$3D` —in `$00-$3F`, where the cartridge has no data— and
its unit. The ROM seeks `MSU_SPILL_AT + unit`, times 256, and a stream
asked for at unit `$FFFF` goes on from where the port was left: that is how
`lz2_grid` and `lz2_backdrop` chain the streams of one piece.

**The boot asks first**, before loading anything: whether there is an
MSU-1, whether the `.msu` is this cartridge's —its signature— and whether
the region is this build's —its own, against `msu_spill_signature`—. If
any of the three fails, it shows a screen that says so, drawn by the
converter with the game's font (`msu_missing.rs` in `rm2k-snes-convert`),
and stays there. Once it passed, the readers do not look again; and the
seek has no cap, because what is read there has no other source.

**What it costs**, in snes9x, where seeking is instant —measured on
2026-10-11 against a twin cartridge with everything in the ROM—: one to
eight frames more per map load, because the LZ2 literals come from the port
one byte at a time through the CPU instead of with `mvn` (map 66 of A
Blurred Line, with the biggest background: 94 frames against 86). In
Mednafen, besides, each seek: a map of Vampires Dawn does zero to five for
its streams —most, three or four—, one for its panorama, and one per sample
of its package that is in the `.msu`. That was not measured frame by frame.

**How it is tested**: A Blurred Line fits, so the suite runs this path on
two cartridges of its own, `test-msu` —everything that may go, in the
`.msu`, with the converter's `--msu-everything`— and `test-msu-rom`, its
twin with everything in the ROM. `test_what_travels_in_the_msu_arrives_the_same_as_from_the_rom`
checks VRAM and WRAM after each map load against the converter, like for
the ROM; `tests/rpgmaker/test_msu_twins.py` compares the boot —with
map 6's panorama— frame by frame, a battle with its battlers, and the
music's envelope, with the samples zeroed as the control; and
`tests/rpgmaker/test_msu_required.py` the screen, without the `.msu`
and with a region from another build. **Not run by the suite**: a big
picture from the `.msu` —no map of `test-msu` shows one—, a light backdrop
from it —with the MSU-1 the battle uses the quality one—, the seeks'
wait, and the console.

## The debug screen

The pause menu's debug screen —see [the pause
menu](the-pause-menu.md#the-debug-screen)— is a tool for testing the port,
and taro tests with the `.msu`; it is not needed to enjoy the game. So
since 2026-11-16 **its bulk travels only in the `.msu`**: its font
precomposed —eight phases of each letter—, the switches' and variables'
names, and the windows' tilemap,
bars, tiles and colours. The ROM keeps the code, the geometry, each
letter's advance and reach, and where each piece is.

**They go in the region of what does not fit in the ROM**, above, in any
game: the converter marks them `.msu_only` instead of `.msu`, and the
packing sends those always, whether the game needs the MSU-1 or not —see
`msu_only_block` in `rm2k-snes-mapdata` and `pack_with_msu` in
`rm2k-snes-cartridge`—. In a game that does not need it, the region holds
only those; in Vampires Dawn, those and what did not fit.

**The pause menu asks each time it is drawn** —`dbg_available`, in
`runtime/debug_screen.s`—: whether there is an MSU-1, whether the `.msu`
is this cartridge's and whether the region is this build's, the same three
questions the boot of a game that needs it asks. If any fails, the Debug
row is not there and End Game is the last. Two seeks each time: nothing
without an MSU-1, and one or two frames each in Mednafen.

**What the screen reads, and when**, into `LZ2_BUFFER`, which is free
while it is open —the map reloads it when the pause menu closes—:

```text
when it opens       the windows' tilemap and bars, and their tiles and
                    colours, which go to VRAM
when a page shows   its names: a hundred switches or variables, their
                    table and their strings, into one of two buffers
when a letter       that letter at that phase, 156 bytes, into a cache of
is composed         them —about a hundred and fifty— taken in turn
```

The names go to the buffer the screen does not show, and a page flip waits
until the text on screen is the last one written: the text layer keeps a
name as the address of its letters, so the page shown must not change
under it. The font cannot go whole —121 KB is almost all the WRAM—, so a
letter is read when the text layer composes it and it is not in the cache:
a seek and a DMA each, in the frame's idle time. **How long that takes on
the FX Pak, where a seek outside its buffer reads the SD card, is not
measured**; in snes9x and bsnes seeking is instant.

Every seek here has the cap `msu_data_seek` has, not the uncapped one of
"What does not fit in the ROM": a page that does not arrive in time shows
no names, a letter that does not shows blank and is asked for again next
time, and the windows go without their tiles. No emulator of the suite
seeks slowly enough to run any of that.

**How it is tested**: the debug screen's tests run with the `.msu` next to
the ROM —`test_debug_screen_golden.py` in the ABL port and
`test_debug_screen_room.py` at the ports' root—, and the walk over every
range compares each page's names and every letter the cache holds with the
`.msu`'s bytes. `test_without_the_msu_1_the_menu_does_not_offer_it` opens
the pause menu without the MSU-1 and with a region from another build.
The rest of the suite runs without the MSU-1, so its pause menu has no
Debug row.

## How it is tested

The test suite runs without MSU-1, so its battle goldens are the light
fallback. `tests/abl/test_msu_data.py` in the ABL port runs both paths
in snes9x and bsnes and compares **each one against its golden**
—`msu-data-rom` and `msu-data-msu`—, and the clean backdrop in WRAM:
with the `.msu`, against that backdrop's bytes in the `.msu` itself; without
it, the light one expanded to 8bpp and doubled, against a golden of its
own, whole. The same with a battle in the
world, whose backdrop is that of the terrain the hero stands on
—`msu-world-rom`, its fallback, and `msu-world-msu`, the quality one—, and
that each backdrop is in its fixed place with the same bytes in the `.msu`
of every cartridge. The fallback is covered by
`test_scene_a_battle_whose_backdrop_is_missing_uses_the_fallback`, in
`tests/rpgmaker/test_battle_scene_drawing.py` in the ABL port.

**The wait is not covered by the test suite**, because the three libretro
cores the suite uses seek instantly, and the harness can't simulate the
delay: it sees memory, not the bus, so it has no way to answer anything
else when the ROM reads `$2000`. What exercises it is Mednafen, by hand: a
cartridge that opens the battle when the game starts —the flags of
`release` with `battle-droid` as page 0—, Xvfb and `xdotool` to get past
the title screen, F5 in the battle, and reading `SCN_MSU` from the `.mc0`.
That is how it was verified on 2026-09-26, and also by playing up to the
droid with the test suite's save loaded into Mednafen's SRAM. Putting it in
the suite would cost one more cartridge and a test that runs in real time
against an emulator with a window; it wasn't done. What still nobody runs
is the cut by the cap, above.
