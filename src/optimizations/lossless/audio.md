# Lossless: audio

## The stream is per note, with repeats

A theme travels as notes, releases and waits, which the 65816 expands
into DSP writes; what a theme plays the same is said once with an
`OP_REPEAT` that goes back and reads the bytes again.

- **Saves**: ROM in the music.
- **Where**: "the per-note format" and "repeats" in `stream.rs` of
  `rm2k-snes/rm2k-snes-audio/`; `irq_music` in `runtime/music.s`.
- **More**: [Audio](../../internals/audio.md).

## A note names its sound in one byte

A note names its sound by an index into the theme's table of sounds, and
a theme with more than 256 pages through them with `OP_SOUND_PAGE`.

- **Saves**: ROM in the music.
- **Where**: `stream.rs` of `rm2k-snes/rm2k-snes-audio/`.
- **More**: [Audio](../../internals/audio.md).

## A single sample bank, uploaded in packages per map

The tracks share one sample per General MIDI program and per drum hit,
and what goes to the ARAM is a package per map: each package with its
effects has to fit in the 64 KB, not the whole bank.

- **Saves**: ROM and the one-byte SRCN; the ARAM is spent on what the map
  plays.
- **Where**: `Bank` and `allocate` in `bank.rs` of
  `rm2k-snes/rm2k-snes-audio/`; `package_load` in
  `runtime/audio_package.s`.
- **More**: [A single sample bank](audio/the-sample-bank.md).

## Effects share places in the directory

Two sound effects that never go in the same package go to the same place
in the SPC700's directory, so the directory needs as many effect places as
the fullest package has effects.

- **Saves**: directory entries, which are one byte's worth.
- **Where**: `effect_places` in `bank.rs`; `record_effect` in
  `runtime/audio_package.s`.
- **More**: [Audio](../../internals/audio.md).

## Identical samples share their entry

Two keys whose samples come out the same —the same BRR and loop— are
stored and uploaded once.

- **Saves**: ROM and directory entries.
- **Where**: `Bank::directory` in `bank.rs` and `merge_identical` in
  `compile.rs` of `rm2k-snes/rm2k-snes-audio/`.
- **More**: [Audio](../../internals/audio.md).

## A package's effects lie in the `.msu` in the order they go up

The sound effects a package takes from the `.msu` are stored there one
after the other, in the order the ROM uploads them, so loading a package
seeks once and then reads straight through; an effect in two packages is
stored twice.

- **Saves**: seeks, which cost one or two frames each in Mednafen.
- **Costs**: `.msu` bytes for the effects that repeat.
- **Where**: `msu.rs` of `rm2k-snes/rm2k-snes-audio/`; `se_msu_upload` in
  `runtime/msu_effects.s`.
- **More**: [The `.msu` file](../../internals/the-msu-file.md#the-effects).

## With MSU-1, the themes with a track upload no samples

A package does not upload the samples of the themes the MSU-1 plays,
unless the track is missing from the card; the effects then get the ARAM
the music left.

- **Saves**: ARAM and loading time.
- **Where**: `song_per_msu` in `runtime/audio_package.s`;
  `rates_per_package` in `effects.rs`.
- **More**: [Audio](../../internals/audio.md), [The `.msu` file](../../internals/the-msu-file.md).

## Boot is sized for the boot package

Boot waits for loading the map, the driver and the directory through the
IPL, and the boot map's audio package, with a margin —not for a full
ARAM—. Since the per-map packages, the IPL uploads only the driver and the
directory, and the samples go through the driver's command 3, three bytes
at a time, so what goes up at power-on is a single package and boot can
be measured against it. It replaced a boot sized for the ARAM ceiling,
which froze the map at power-on for almost two seconds.

**Each cartridge carries its own** `BOOT_FRAMES`, since 2026-10-11: the
estimate for its boot package, or `BOOT_FRAMES_FLOOR`, whichever is
larger. The floor keeps a cartridge with a small package on the frame it
always started on, with a byte-for-byte identical ROM; it lives in the
runtime and not in each game's project because what it protects is the
runtime's —the loop starting on the same frame whatever small data goes
in—. The only ceiling left is the byte the ROM counts boot frames in.

- **Saves**: frames at power-on.
- **Costs**: if the boot map's package grows, that cartridge's boot grows
  with it, by itself. The tests read it from the symbol table, so it moves
  no golden.
- **Where**: `boot_frames` in `rm2k-snes/rm2k-snes-audio/src/emit.rs`,
  which `rm2k-snes-audio cartridge` writes into the cartridge's
  `musicdata.inc`; `BOOT_FRAMES_FLOOR` in `runtime/far_ram.s`.
- **More**: [Audio](../../internals/audio.md).

## A package uploads only what the ARAM lacks

When a map's package loads, the samples it shares with the one before
stay where they are and only the missing ones go up. Where each one goes
is decided first and the upload comes at the end, so when the gaps leave
something out and everything is placed again from scratch, nothing is
uploaded twice; and if what the package wants does not fit even in an
empty ARAM, that rebuild is not tried, because it would only upload again
what was already there.

- **Saves**: loading time —the frozen frames of a map change, of entering
  and leaving a battle, and of a `PlayBGM` that completes its package—.
- **Where**: `package_load_done`, `place_sample`, `upload_placed` and
  `wanted_fits` in `runtime/audio_package.s`.
- **More**: the comments in `package_load_done`.

## Samples go up three bytes per handshake

The driver takes a block of up to 85 triplets with one command, three
bytes per token, and the 65816 reads the next triplet while the SPC700
stores the one before, which is what takes longest; only the last one or
two bytes go one at a time.

- **Saves**: loading time —a third of the handshakes, and the reads
  hidden behind the SPC700's—.
- **Where**: command 3 of `DRIVER` in
  `rm2k-snes/rm2k-snes-audio/src/spc700.rs`; `upload_sample` in
  `runtime/audio_package.s`; `msu_a_aram` in `runtime/msu_effects.s`.
- **More**: the comment on `DRIVER`.

## A note goes to the voice its channel last held

When it can choose, the voice allocation gives a note the voice that
last played its channel: same instrument and same volume, so the note
needs no volume in the stream and the decoder's register cache skips
what it would write again.

- **Saves**: ROM in the music —such a note is two bytes— and DSP writes.
- **Where**: `free_order` in `rm2k-snes/rm2k-snes-audio/src/voices.rs`;
  `ev_note` and `mus_write` in `runtime/music.s`.
- **More**: the comment on `free_order`.

## A teleport's package goes up in the fade's frozen ticks

Inside a teleport's fade, the map's package is placed when the map loads
but its samples are not uploaded there: only the song the map turns on
goes up at once, when `music_start` asks for it, and the rest is uploaded
block by block in the ticks of the fade that follow, which are frozen and
idle, stopping each one before the vblank and before the music's IRQ.
On the last black tick, what of the music the ramp up could not take goes
up at once, still in the black, so no tick of the ramp lasts longer; and
the fade does not end until the package's music is complete. Nothing can
sound in between —the world is frozen and the effects of the map left
behind were silenced—, and the ARAM and the directory end up the same.
The sound effects do not hold the fade: see the next one.

- **Saves**: loading time. Measured on 2026-10-18 over 200 of A Blurred
  Line's door pairs without MSU-1, the uploads were 17 of the 30 frames a
  load added to the fade; the fade's 43 idle ticks take some 26 KB of
  them.
- **Costs**: nothing seen or heard. The song the map turns on still goes
  up in the load, because it starts there, as in RM2k.
- **Where**: `upload_placed`, `package_pump`, `pump`, `pump_room`,
  `package_settle`, `whole_ones` and `song_now` in
  `runtime/audio_package.s`; `LOAD_DEFER` and `fade_tick` in
  `runtime/enter_map.s`.
- **More**: the comments in `upload_placed` and `package_settle`.

## A teleport's sound effects go up when they are needed

A teleport's package places its sound effects with the rest, but the fade
does not wait for them: they go up block by block in the idle end of the
game's ticks once the map is playing —`package_idle_l`, from the main
loop, stopping before line 180 and never in a tick that overran—, and an
effect asked for while it still waits goes up whole right then, before
it starts (`effect_now_l`, from `sound_play`). The ones from the `.msu`,
which only go up whole, wait for that moment. A new package lets go of
what the previous one left waiting instead of uploading it. A `PlayBGM`
that rebuilds the same package keeps its effects waiting; any other load
outside a fade —a battle's, the boot's, a save's— uploads everything at
once, as before. And **the package that is already in ARAM, whole,
uploads nothing**: a jump within the same map, which with the `.msu`
uploaded every effect again.

- **Saves**: the black stall of a jump between two maps whose packages do
  not fit together. The case is The Frozen World's battle, whose every
  turn jumps from the command grid, map 3, to the battle screen, map 1,
  and back: measured on 2026-11-22 over the game's own turn, the two
  jumps went from 81 and 57 frames over their ticks to 33 and 39 without
  MSU-1, and from 122 and 124 to 33 and 37 with the `.msu`; the black
  tick of each, from 64 and 33 frames —105 and 102— to 16, which is the
  map's load. See
  [`frozen-world-every-turn-re-uploads-the-battle-screens-audio`](https://github.com/lautarodragan/the-advocate-work/blob/main/tasks/frozen-world-every-turn-re-uploads-the-battle-screens-audio.md).
- **Costs**: an effect that plays before its turn freezes the picture
  while it goes up —a few frames for one from the cartridge, up to some
  fifteen for a 22 kHz one from the `.msu`—, with the music going on; and
  with the `.msu`, that effect's seek, one per effect instead of one per
  package.
- **Where**: `upload_placed`, `package_drop`, `package_idle_l`,
  `effect_now_l`, `pump` and `PKG_WHOLE` in `runtime/audio_package.s`;
  the calls in `sound_play`, in `runtime/sound_effects.s`, and in the main
  loop, in `runtime/boot.s`.
- **More**: the comments in `upload_placed` and `package_load_done`.
