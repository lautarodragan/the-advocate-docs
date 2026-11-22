# The test game

> **A progress page, kept up to date.** Born on 2026-10-15 with the test
> game's skeleton (Calandria), when phase 2 of
> [the research](../research/2026-10-02-the-test-game.md) started. Whoever
> moves a test into the test game, or adds to it what a test needs, updates
> **Where it stands**. The task is
> [`the-test-game-its-skeleton`](https://github.com/lautarodragan/the-advocate-work/blob/main/tasks/the-test-game-its-skeleton.md).

The engine's ROM tests —the ones that build a cartridge, run it in an
emulator and compare what it draws— live in A Blurred Line's suite, because
until now there was no other game to run them on: most of them test what
RPG Maker 2000 does, using that game as a stage. That ties the tools to a
game that is not ours to publish, and a tool change is only verified by
running another repository's suite. The test game is a game of our own,
in the-advocate, with free assets, for those tests to come back to.

It is here, under progress and not under internals, because what matters
about it is how far it has got: it grows a test at a time, and each one
that moves out of A Blurred Line changes this page.

## Where it stands

**The skeleton**, 2026-10-15: one map, the hero and one NPC, and its first
tests.

* **The game** is `test-game/` in the-advocate: a Ditto exact dump written
  by hand —liblcf's names, enums by name, defaults left out— and each map's
  tile layers as a **sketch**, a grid of characters with a legend of the
  tiles each one stands for. `rm2k-sketch`, a new tool in `rm2k/`, makes a
  sketch into the dump's `.tiles`; `ditto pack` makes the RM2k files, which
  open in the editor; and the cartridge is built from them like a port's,
  with the installed tools.
* **The assets** are EasyRPG's RTP, unchanged: the Exterior ChipSet (Jason
  Perry, CC0), the People1 CharSet (VictorSena, CC BY 4.0) and the System
  (andwhyisit, CC BY 4.0). Its `AUTHORS.md` carries each one's line, and a
  test of the tools fails if a picture has none. The System is the RTP's
  because nothing can be drawn in a window without one, and the console's
  limits on it are the converter's to meet: it merges what does not fit and
  says so in the build.
* **The cartridge** is built with no RTP —an empty `--rtp-dir`— and without
  a title screen, and the SoundFont comes in `SOUNDFONT` like for the audio
  tests: once the game has music, its samples go into the cartridge, so it
  has to be a free one.
* **The test**, `test_meadow.py`: the cartridge boots on the meadow, the
  hero walks up to the elder, A starts his page and his message is
  composed; two goldens, the map and the message, in snes9x and in bsnes.
  The harness was a small one of its own and the core driver a copy of A
  Blurred Line's, until the harness's generic part came over on
  2026-11-22: see below.
* **The cases**: until 2026-11-22, `test-game/cases/`, one JSON per test
  page, like a port's, which the harness chose by the case variable before
  the first game frame, the way A Blurred Line's does. Since then they are
  the game's own: see "Playable by hand" below.
* **What moved out of A Blurred Line**: switches past 1023, on 2026-10-15
  —`test_switches_past_1023.py`: a page conditioned on 1100, a
  `ControlSwitches` and a branch on it, and the page not running before
  its switch is on—. In A Blurred Line it needed a fork of its map 348,
  since that game names no switch past 1008, and the cartridge taro plays
  carried it; here the meadow carries the event, invisible. The task is
  [`switches-past-1023-tested-in-the-test-game`](https://github.com/lautarodragan/the-advocate-work/blob/main/tasks/switches-past-1023-tested-in-the-test-game.md).
* **What was born here, on 2026-10-15**: the faces of the message window
  —`test_faces.py`, the case `faces`: Actor1's face from the RTP on the
  left, on the right and mirrored, kept over a box with no command, changed,
  and cleared; five goldens—. None of the three games that show faces has a
  page that does all of it, and A Blurred Line shows none. The task is
  [`draw-the-faces`](https://github.com/lautarodragan/the-advocate-work/blob/main/tasks/draw-the-faces.md).
* **And on 2026-10-16, the first picture**: the lantern, map 5, where a
  parallel event lays `Picture/Night` —a near-black mask with a cone of
  light, ours, drawn by a script— over the map at transparency 35, fixed
  to it, the way Desert Nightmare lights its interiors; the project has
  `[graphics] map_pictures`, so it is drawn into the map, its layer above
  included —`test_map_picture.py`, the case `lantern`: in the dark, and
  under the lantern with the camera scrolled; two goldens—. A Blurred
  Line has the knob off, and Sunset Over Imdahl's maps are paintings, so
  the ChipSet side of the rule runs only here. The task is
  [`pictures-that-go-over-the-map-outside-sunset`](https://github.com/lautarodragan/the-advocate-work/blob/main/tasks/pictures-that-go-over-the-map-outside-sunset.md).
* **And on 2026-11-16, the vblank's budget**: the terrace, map 9, a slab of
  grass in a night sky of ours, `Panorama/Stars`, that scrolls on its own,
  with walkers under a transparent box in the middle of the screen
  —`test_terrace.py`, the case `terrace`—: a stress case of A Blurred
  Line's map 1 that there would have moved the text block and the goldens.
  It asks, on a second cartridge built with the vblank's probe, on which
  line each of the box's lines goes up and that none waits two vblanks;
  on bsnes's accurate PPU with the display-access audit, that nothing
  reaches the screen while it is drawn; and two goldens. The first of the
  engine's timing tests that lives here. The task is
  [`the-test-game-a-transparent-box-over-a-scrolling-panorama`](https://github.com/lautarodragan/the-advocate-work/blob/main/tasks/the-test-game-a-transparent-box-over-a-scrolling-panorama.md).
  The same day the scene went on in the rain, which that map has on too:
  an event at the hero's side starts it, strong, when talked to, and four
  boxes more are measured the same way under it, with two goldens more
  —[`the-test-game-s-terrace-in-the-rain`](https://github.com/lautarodragan/the-advocate-work/blob/main/tasks/the-test-game-s-terrace-in-the-rain.md)—.

**And on 2026-11-22, the end of phase 2**: the harness's generic part and
the first ten tests. What runs a cartridge in a libretro core and asks it
questions knows no game —the driver and the cores' patches, the cartridge
as the builder answers for it, the tick clock, the goldens at 15 bits,
RPG Maker's rules, the engine's numbers—, and it is one Python package
now, `rm2k_snes_harness` in `rm2k-snes/harness/` of the-advocate, beside
the runtime whose symbols it reads. The test game uses it as it is, and
its stopgap harness and its copy of the driver are gone; A Blurred Line's
harness, and the ports' other games, take it from where
`rm2k-snes-project --harness` says —the checkout the tools were installed
from, like the runtime— and keep on top of it what is theirs: that game's
title screen, its menus, its battles, its walkthrough. Then ten of its M1
files moved: the variables, `ConditionalBranch`, the labels, the chained
teleports, a door to the same map, the hero's speed, `KeyInputProc`,
`StoreTerrainID`, the action key on the hero's own cell and the tilemap's
window against the camera, with their cases rewritten on this game's maps
and run in snes9x and in bsnes, where that game ran them in snes9x alone.
What their maps gave them is built here: the lookout's dispatcher, a
terrain for each of the showcase's first four tiles, the doors —two rooms
of one map crossed by teleports to itself— and the patchwork, stripes no
two neighbouring rows draw alike. They ask the RAM, not the screen, so no
goldens came with them. The task is
[`the-test-game-takes-the-harness-and-the-first-ten-tests`](https://github.com/lautarodragan/the-advocate-work/blob/main/tasks/the-test-game-takes-the-harness-and-the-first-ten-tests.md).

**And the same day, what no test looked at**: `EraseEvent`'s three
runtime paths —the erased event's row hidden and its route let go, its
pages never met, a parallel page cut where it erases its own event— were
checked only by playing Desert Nightmare. The erasure, map 14, has a
statue on the hero's layer and a walking pacer with a parallel page, both
erasing themselves, and `test_erase_event.py` asks the screen whether
they are still drawn —their cells against the same grass three cells
over—, and the RAM whether the hero walks through, whether the pacer
still walks and whether what follows the erase ran; and what A Blurred
Line's test asks of an invisible event —the foreground page goes on, the
other pages do not start, a load of the map brings it back—, with a jump
to the same map that does not. Each path was seen red broken. The task is
[`eraseevent-no-test-looks-at-what-is-seen`](https://github.com/lautarodragan/the-advocate-work/blob/main/tasks/eraseevent-no-test-looks-at-what-is-seen.md).

**And on the same day, playable by hand**: taro asked for a build where
everything the tests reach can be reached by playing, and then that the
tests play it too —"mocking is a code smell"—. The meadow is the hub: a
signpost, the guide, teleports to every map; a board right below where the
hero starts, the tester, runs every case from `ShowChoice` menus, with the
hero where the harness had them at boot; an item, the Way Home, brings the
hero back from anywhere. The cases moved from `cases/` to `tester/`, and
`hub.toml` says the menus, which `just hub` writes into the meadow. The
tests boot and walk to the signpost or the board, and keep a shortcut only
where playing cannot get there, which each says on its line. `just
deliver` leaves the build for taro. The hub is on the meadow and not on a
map of its own because the converter puts test pages on the map the
cartridge boots on, and half the cases run against the meadow's events.
The task is
[`the-test-game-playable-by-hand`](https://github.com/lautarodragan/the-advocate-work/blob/main/tasks/the-test-game-playable-by-hand.md).

Of the seven assumptions about A Blurred Line the research found in the
tools when it first wrote a game from scratch, four are gone —a debug map
the project had to carry, a single-map game not building, a fallback
backdrop that had to exist, a System with too many colours for the
window—; one is still there —the cartridge has to be called `release`—,
and two were not measured again, because the way the game is built does
not reach them. The task says which.

## What is next

In the order the research gives, each a task of its own:

1. **The rest of the harness**: what A Blurred Line's still keeps that is
   not that game's —its menus and screens, its models of the battle and
   the music, its `Session`'s clock, which the generic one copied—, into
   `rm2k_snes_harness` as the tests that need it move. What stays with A
   Blurred Line is its walkthrough and its places.
2. **The cases**: the rest of `tests/rpgmaker/` in A Blurred Line —phase
   3, the rest of M1— moving over one by one into the test game's
   `tester/`, each offered by the tester's menus and with goldens born anew —and looked at once, since a new
   golden defends whatever the ROM drew that day—.
3. **What the moved tests need**, as they need it: more maps, a title,
   battles —with backdrops and enemies made for it—, music from
   public-domain scores.
