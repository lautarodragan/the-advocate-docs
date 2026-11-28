# The pipeline

What each step does and why the tools read a JSON dump is in [From a game
to a ROM](../pipeline.md). This is the same road with the names of the
crates that do each step and what each one leaves behind; from the dump
on, it is all one process, `advocate build`, calling them as libraries:

```text
--game-dir (.lmu .ldb .lmt)       --advocate import, with Ditto-->  data/ (JSON)
data/ + patches/manual/
+ images, MIDI and WAV            --the converter, rm2k-snes-convert--> build/<rom>/
  (--game-dir, --rtp-dir)           (its stages compiled in: wally,   mapdata.inc, musicdata.inc,
                                     events, screens, images,
                                     battle, mapdata, patches, audio)
                                                                  <rom>.msu (battle backdrops),
                                                                  patches/ (decisions),
                                                                  degradations.json
runtime/*.s + those .inc    --the cartridge library's assemble-> <rom>.sfc + <rom>.sfc.sym
                                    (bank packing, ca65 + ld65)   and, with [rom] msu = "required",
                                                                  what does not fit, behind <rom>.msu;
                                                                  space.json, what it has left
<rom>.sfc + MIDI or replacements  --rm2k-snes-audio's tracks----> <rom>-N.pcm
                                    (fluidsynth, ffmpeg)
every cartridge                   --advocate build, at the end--> build/advocate.json
```

The paths on the right belong to the game's project —where its
`advocate.toml` is, which declares them; `build/` is always beside it—,
and `<rom>` is its `[rom] name`. `advocate build` does all of that for
each cartridge the project declares, and leaves `build/advocate.json`:
the project as the tools read it, the machine values it was given, each
cartridge with its cases, whether it is up to date and its space, the
versioned patches compared, the memory map and the runtime's sources,
which is what a game's tests and recipes read instead of asking a
binary. Until 2026-11-25 the build was `rm2k-snes-cartridges`, which ran
`rm2k-snes-convert` and `rm2k-snes-audio msu` as processes.

How A Blurred Line calls it, with which recipes and which flags: [its
pipeline](../ports/abl/pipeline.md). In which order the converter's
stages decide, and which budget each spends: [the converter's
stages](the-converters-stages.md).

## What the cartridge does with less than RM2k

Next to each ROM the converter writes `degradations.json`: **every place
where the cartridge runs something with less than RM2k does** —a command
accepted without being done, a parameter read and dropped, a graphic
change skipped, a teleport to a map the cartridge does not carry turned
into a message box, a ChipSet animation cut to fit, a map's rain or
battle animations not drawn—. A page left out whole is not in it: that is
`culprits.json`, beside it, which says which commands keep each page out.
Nor are the colours the converter merged,
which are the patches' (`patches/`, above).

It is **complete by construction**: each entry is noted where the loss
happens, not kept in a list by hand. The event compiler notes the
commands, in the arm of `compile.rs` that compiles each one, and its
`losses.rs` keeps where it is; Wally notes what a map does not draw, the
emission what it cut, the screens what they cut or merged, the converter
its own decisions —the events left out for RAM, a hero's flash on a shared
palette—, and the audio stage what a package that did not fit in the ARAM
lost, which it answers beside its files —and leaves in its `audio.json`
under `losses` too—; `degradations.rs` in `rm2k-snes-convert` gathers them
and writes the file after the music, which runs last. A page that does not compile takes its losses
back, as it takes its texts back, so what is listed is what reaches the
cartridge.

Each entry has a `kind`, a stable slug to count by —`system-bgm-not-changed`,
`route-repeat-dropped`, `tint-gradient-dropped`…—, a `text` that says what
is lost, and where, as precisely as whoever lost it knows:

- **a map's page**: `map`, `event`, `page` (RM2k's numbers), the event's
  cell as `x` and `y`, and the index of the `command` in the page, with
  its `code` and `name`; `page` is null when a whole event is lost;
- **a pasted body**: a common event, or another event's page, that a
  `CallEvent` pastes into the page goes in `via`, with the command each
  one was at, under the call's command. A `CallEvent` compiled as a call
  —`OP_CALL`, `OP_CALL_PAGE`— has no `via`: the callee's losses are its
  own, under the common event or the page it is, and the call's own loss,
  if any, is at the call —a callee with no code, `call-skipped`; a chain
  deeper than the console's call stack, `call-stack-exceeded`—;
- **a common event** that runs as a shared body or a parallel one:
  `common` and its `command`, and the file's `commons` says how it runs
  —its trigger and switch— and which pages call it. An automatic one that
  shows small pictures is also copied into the maps where the copy draws
  them, as the pages of an added last event (`common_pages.rs` in
  rm2k-snes-convert; see [the converter's stages](the-converters-stages.md)): what
  the copies lose is not listed again, it is the common event's;
- **a troop's page**: `troop` and `page`, and its `command` in RM2k's list;
- **a map**: only `map`, and `events` when it is about some of them;
- **the cartridge**: no place, for its screens and its battle text.

The list is sorted that way —the maps by map, event, page and command,
then the common events, the troops and the cartridge— and `counts` says
how many of each kind there are. The converter prints the total as it
writes it.

## When a cartridge is rebuilt

`advocate build` rebuilds a cartridge only when its stamp
—`stamp.txt`, next to the ROM— no longer matches: the cartridge's converter
flags, its test cases, and a **fingerprint** of everything else that goes
into the ROM. The fingerprint looks at contents, never at dates:

- the whole `runtime/`, and the binaries that build: `ca65`, `ld65`, and
  the running `advocate` —the converter, its stages and the assembler are
  libraries compiled into it—, not whichever is first on the PATH;
- the game's dump, `data/`, whole, and its hand-made patches —with each
  name, for the files of `patches/manual/`'s RM2k folders, below—, its project
  file and the files that file names —custom menus, its own font, its
  ExFont's sheet—;
- **the original game's resources**: every file below `--game-dir` with an
  extension the stages read —`.png`, `.bmp`, `.xyz`, `.mid`, `.wav`—, with
  its name. Not the `.lmu`, `.ldb` and `.lmt`, which reach the ROM through
  the dump, nor the saves, which change whenever someone plays the
  original. It is what the stages *can* read, not what they read: a
  resource the game ships and never uses goes in too.

**A game or an RTP that is a `.zip` goes in by the `.zip`'s content**,
whole: hashing even the biggest of the ported games takes tens of
milliseconds.

**The RTP and the SoundFont, as directories, go in by their path, not
their content**:
they are not edited, and hashing them would read them on every run. If one
is changed in place, `advocate build --force` rebuilds the cartridges
named —or all of them— without looking at the stamp. The MSU-1 tracks do
not need it: they are outside the stamp, and their own cache keys on the
SoundFont's content.

Without `--game-dir` the fingerprint is an error, not a guess. **Whether
a cartridge is up to date is what the last build saw**: `advocate build`
writes it into `build/advocate.json`, and a game's tests read it there
instead of computing the fingerprint again —every recipe that runs tests
builds first, and a `pytest` run by hand after editing a source runs the
ROM of the last build (taro's, 2026-11-25)—. Until that day the tests
asked `rm2k-snes-cartridges --list --json`, with the machine flags of
building.

## The conversion cache

What a build computes and another can reuse goes into one directory,
`--cache` —by default `cache/` in the project's build—: each stage's
answer, `ca65`'s objects, the volume ceilings, the MSU-1 tracks —hard
links to the build's `.pcm`, so they cost nothing while a build keeps
them—, and the games unpacked from their `.zip` and with the port's own
files on top, below. Each value is a file `<category>/<key>`, and its key
carries the contents of what it was made from and with —the inputs, the
binary that made it—, never a checkout's paths, so a machine can point
every checkout's `--cache` at the same directory and a new worktree finds
what another one already built. A value is written aside and renamed into
place, whole or not at all, because two processes may make the same one
at once; and `NO_CACHE=1` is how a build is checked against it: with and
without, the ROMs must come out byte for byte equal. See `cache.rs` in
`rm2k-project`.

**What nobody used in two weeks is deleted** (taro's, 2026-11-28):
since a key carries the binaries that made the value, each new converter
leaves the whole previous set of answers behind, and on that day the
machine's cache had reached 92 GB in five days. Each use of an entry
—reading it, fetching it as a hard link, a build finding its own copy of
a track still the cache's— stamps it by its **access time**, written by
the cache itself, because the kernel's is not to be trusted —`relatime`
writes it once a day at most, `noatime` never, and a hard link is not
even read—; not by its modification time, which a track shares with the
build's `.pcm`. And once a day, `advocate build`, before touching the
cache, deletes every entry whose stamp is more than fourteen days old,
and what a process that died left half written.

**Nothing is deleted while a process that may read it lives.** Every
process that uses the cache holds a shared lock on its `.lock` until it
exits —`Environment::unpack` takes it too, so a tool reading the game
below the cache holds it even when it unpacks nothing—, and the sweep
needs it exclusive: a build that finds the cache held by anyone else
does not sweep and does not wait, and the next build will. Locking each
entry would let the sweep run beside other builds, but nobody would
reliably unlock what a crashed process held; a lock that goes with the
process is released however it dies.

## A game or an RTP as a `.zip`

`--game-dir` and `--rtp-dir` take a directory or a `.zip` of one, with the
game's files at the root of the archive. Nobody reads inside the `.zip`:
the lookups are several, in several processes, and some hand a path to
`fluidsynth` or `ffmpeg`. Instead, every tool that reads the game calls
`Environment::unpack`, in `rm2k-project`, right after reading its flags;
that unpacks the `.zip` once into `zip/` in the project's cache, named by
its content, and from there on the value is that directory —also for the
tools it runs—. A tool that forgets to call it gets an error, not an empty
game: a lookup that finds nothing in the game falls back to the RTP without
a word.

`advocate import` unpacks it into the same `zip/`, but not through
`Environment::unpack`, which also puts the port's own files in front of
the game's (see below): what is imported is the game as its author left
it. A game's tests take the directory a build read it from, the
`machine` of `build/advocate.json`, the same copy. Until 2026-11-25 both
asked `rm2k-snes-project --unpacked game_dir`, and until 2026-11-27 a
game's `dump` recipe read the `.zip` itself, with a script of the
ports.

A `.zip` is what keeps the original game untouched: what the port changes
goes in its patches, not in the game.

## A port's own files, in front of the game's

The RM2k folders of `patches/manual/`, in the game's project
—`Picture/`, `ChipSet/`…—, hold files that replace the game's with the
same folder and name without extension, without regard to case —see [A
port's own files replace the game's, by
name](../decisions/a-ports-own-files-replace-the-games-by-name.md)—. What
else lives in `manual/` —the maps, the `NNNN.json` records, the common
events— is not a file of the game, and is skipped here: see [The manual
patches are one folder per kind of
asset](../decisions/the-manual-patches-are-one-folder-per-kind-of-asset.md).
`Environment::unpack` does it too, right after the `.zip`: it builds the
game with ours on top in `own-files/` in the project's cache —links to the
game's files, ours copied—, named by the game's identity and by our files'
names and contents, and from there on `--game-dir` is that directory, also
for the tools it runs. A copy is never half of two versions: it is written
aside and renamed, like the `.zip`'s.

The directory carries `.rm2k-own-files.json`, what it replaced: a tool that
receives one already built keeps it, and the converter prints it in
`converter.log`. `build/advocate.json` names the same directory as the
one the game is read from, so a game's tests see the port's files too.

## A port's own common events, in the dump

`patches/manual/commonevents/NNNN.json` is a common event of the game,
redone: the whole record as Ditto dumps it, with its id inside, and it
replaces the game's of that number —a number the game lacks is refused,
since nothing would call it—. The common events are not a file of the
game but a section of the dump, and five stages read them on their own
—the converter, the event compiler, Wally, the screens and the music—,
so the converter does what `Environment::unpack` does with the files: it
asks Ditto for a dump that is the game's with that section replaced
(`Dump::with_section`, every other file a link to the game's), in
`build/patches-manual/dump-<hash>/` of the project, named by the game's
common events and ours, written aside and renamed; and from there on
`--data` is that dump for every stage it calls. See
`fork::with_common_events` in rm2k-snes-convert. Frozen World's B menu
was the first, on 2026-10-19.
