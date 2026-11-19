# What it takes to be playable (2026-11-19)

> **Measured on 2026-11-19** by Hurón, on Desert Nightmare's `release`
> built from the-advocate-ports `1ea90bf` with the tools of the-advocate
> `62b6e5cb`, played in snes9x through the ports' harness. It is a
> snapshot: the numbers are from that day and this file is not updated.
> How each one was obtained is at the end, in "How it was measured". The
> task is
> [`desert-nightmare-what-it-takes-to-be-playable`](https://github.com/lautarodragan/the-advocate-work/blob/main/tasks/desert-nightmare-what-it-takes-to-be-playable.md).

taro, translated: "That game surprised me a lot —it looks very good, but
it is practically unplayable in our current port." This is why, played
as far as it goes and compared with RPG_RT, and what would fix each
thing, ranked.

## Summary

* **The story stops on day 1, three times over.** Played from the boot
  —title, "With Intro", the motel, the street, the bar, Jake and the dog,
  the gate, the alley, the box, the warehouse— it stops **at the box on
  map 12**, which cannot be climbed; past it by hand, **at the warehouse's
  lamp**, because no item can be used; and the next step, **the
  warehouse's safe**, is a number lock that does not compile. Each of the
  three is a limit of ours, and the first two are small.
* **What makes it unplayable where it runs is how the game is built**:
  its towns are a painted panorama with **hundreds of events on top**,
  each a 24x32 piece of a building cut from a CharSet. The console draws
  every one as an object, so on the ~20 town maps **the buildings are
  missing or white rectangles**, the people further back are cut, and
  **the game runs at half speed** —2 to 2.65 frames per round of the main
  loop—. On map 2, with nothing drawn, it is back at one.
* **The game's own menu draws nothing.** Inventory, keys, notes and maps
  are 155 pictures of 320x240 in common events; B opens the menu and the
  map stays on screen. Items are chosen blind, and the notes —where the
  game writes its safe codes— cannot be read.
* **Messages have no box on 175 of the 189 maps**: the box is two
  pictures that fit on 14, and the map's upper layer cuts the text.
* **What is the console's** is the objects —128, 32 on a line, 34 slivers
  of 8 pixels— and the VRAM their poses take; **drawing the scenery as
  objects is RM2k's model, carried over**, and SNES games draw it in the
  background. Everything else above is ours.

## How far it plays

From the boot, tapping A through scenes as `tests/test_boot.py` does, and
walking with `play_script.py`'s floor plan:

    stage                          map        tick    what happens
    title, "With Intro"            1          0       fine, over the burning face
    the intro                      …→16       11,404  control in the motel
    the map at the desk            16         11,696  obtained; the second box has no frame
    out to the street              2          11,834  no buildings drawn; 2.0 frames a tick
    the gate, "Damnit!"            2          12,962  switch 219
    the bar, Jake                  3, 19→2    15,199  Jake chases the dog (229)
    the gate opened                2          15,241  (13,0) leads nowhere; (12,0) does
    the alley, the dog's scene     4→11→12    23,873  the dog's dragging pictures not drawn
    the box pushed to the wall     12         24,286  the climb does nothing: STOP 1
      (the box's row moved by hand)
    the blood trail, back          13→12→4    27,360  Alan's scene (231, 204)
    the gate to the warehouse      14→15→21   32,384  map 14 at 2.0
    the bulb                       22→23      33,157  obtained
    the bulb at the lamp           26         33,866  nothing: STOP 2
    the safe (not reached)         28                 does not compile: STOP 3

The intro is the slowest stretch to sit through only because it is long;
it plays right. The title and the start menu are fine: the garbage the
port's "Known issues" still names is not seen. **No map hangs the ROM**:
a door's teleport into each of the 189 and 240 ticks later, every one is
alive; the four that looked hung in the sweep run from a fresh boot, and
map 147 is a death scene that ends in the title, as it should.

## What breaks, and why

### 1. A second event on a cell is never asked (stops day 1)

The box puzzle on map 12: pushed against the wall the box (event 81)
stands on (12, 4), the same cell as event 87, the touch page that climbs
to map 13. RPG_RT walks **every** event on the cell and fires each whose
trigger matches —easyrpg's `Game_Player::CheckEventTriggerThere`—; the
console's `event_in_cell` (`runtime/action_key.s` in the-advocate)
returns the first row on the cell and stops. The box comes first, its page
has nothing for a bump, and the hero stays. **With the box's row moved off
the cell by hand, the same bump climbs.**

The gate on map 2 is the same thing with a twin: three identical
teleports on (12, 0), (13, 0) and (14, 0); the invisible dog, sent to
(13, 0) by Jake's scene, has a row there, so the middle cell —the
straight way through— does nothing, and only the sides work.

*Ours*: the runtime's lookup. Draft:
[`a-second-event-on-a-cell-is-never-asked`](https://github.com/lautarodragan/the-advocate-work/blob/main/tasks/a-second-event-on-a-cell-is-never-asked.md).

### 2. Items cannot be used (stops day 1)

Common event 20, "Sache benutzen", is how every item is used: it reads
the hero's map, cell and facing, and turns on the switch the target
waits for. It does not compile, nor does 14, the maps screen: both read
the hero's **map id** with `ControlVars` (operand 6, field 0), which the
event compiler leaves out because A Blurred Line never asks
(`VAR_CHARACTER` in `compile.rs`). Played: the bulb in hand, at the lamp,
B, Inventory, the bulb —switch 6 on, variable 8 at 1— and 242, "the lamp
now works", never comes.

*Ours*: one field of an operand. Draft:
[`desert-nightmare-items-cannot-be-used`](https://github.com/lautarodragan/the-advocate-work/blob/main/tasks/desert-nightmare-items-cannot-be-used.md).

### 3. The number locks (stop day 1 and day 3)

The only four map pages out of 6,773 are the game's locks: the
warehouse's safe (map 28, **71456**), a chest on day 2 (map 42), the
tailor's door (map 123, 1759) and a chest in the second vault (map 156,
**136264**). All four put the `InputNumber` after the message has
closed, a window with only the digits, which the ROM does not have; two
compare against numbers a sixteen-bit variable cannot hold —RPG_RT's go
to 9,999,999—.

*Ours*: a window kind and the variables' width. Draft:
[`desert-nightmare-the-number-locks-do-not-open`](https://github.com/lautarodragan/the-advocate-work/blob/main/tasks/desert-nightmare-the-number-locks-do-not-open.md).

### 4. The menu draws nothing

B opens the game's menu —common 4, a parallel `KeyInputProc`— and its
pages work: switch 5, then 6, then variable 8 with the item under the
cursor. None of its **155 pictures** is drawn —`Introauswahl` behind,
`Optionen1..5`, `Inventar`, `Gegenstand1..`, `Schlussel1..`, `Notiz1..`,
`KarteG1..`, the cursors—: each is a full 320x240 canvas, mostly
transparent, shown by common events; `degradations.json` counts 503 of
its 584 `picture-not-shown` in those commons. The console keeps one
large picture at a time in BG1 and hides the map and the small ones under
it ([What large pictures do not model](../../../divergences/what-large-pictures-do-not-model.md)),
and the commons' small pictures go in sets that fit on few maps
([More pictures at once](../../../research/2026-10-16-more-pictures-at-once.md),
options 4 and 5).

*Partly the console's* —one background layer for a big picture in mode
1— *and partly ours*: the 320x240 canvases are a way of placing icons
that a menu described as data does not need, which is what A Blurred
Line's portable console became (`abl/menus/console.toml` in the ports).
Draft:
[`desert-nightmare-the-game-s-menu-draws-nothing`](https://github.com/lautarodragan/the-advocate-work/blob/main/tasks/desert-nightmare-the-game-s-menu-draws-nothing.md).

### 5. The towns: buildings missing, white rectangles, half speed

**How the game draws a town.** Map 2's panorama, `Eingangsbereich`, is
368x272 —the map's own size, 23x17 cells— and has white holes where the
motel, the store and the gate go; those are **83 events**, one per cell,
each a 24x32 piece of the CharSets `Motel` and `Allzweckladen`, on the
hero's layer. Over the whole game, **3,977 events draw something, and
3,368 of them never change their look and nobody moves them**; 2,386
have one page and no commands at all.

**What the console does with them.** Each is an object, with a pose in
the sprite cache:

* **Not drawn.** On 34 maps "up to N events are on screen at once and the
  cache has M slots" —80 on screen and 15 slots on map 2, 90 and 23 on
  map 5, 74 and 11 on 172—: the ones drawn last wait for a slot, and do
  not come in 240 ticks. On 9 maps the drawn table holds 104 of 114 to
  154; on 7 the RAM's 120 rows leave events out. Where the panorama has a
  hole under them, **the hole shows white**: maps 3, 4, 5, 12, 14 and
  62 in the captures, entered and played. The draft
  [`desert-nightmare-roofs-left-out-show-white`](https://github.com/lautarodragan/the-advocate-work/blob/main/tasks/desert-nightmare-roofs-left-out-show-white.md)
  saw four of them.
* **Cut.** The PPU dropped objects on some line on 59 maps, on most of
  the towns every frame (the debug screen's `Time over`, read from RAM):
  already measured
  for 53 maps by
  [`desert-nightmare-characters-cut-on-crowded-lines`](https://github.com/lautarodragan/the-advocate-work/blob/main/tasks/desert-nightmare-characters-cut-on-crowded-lines.md).
* **Slow.** 18 maps take 2 frames or more per tick —2, 3, 4, 5, 6, 11 to
  14, 43, 59 to 62, 64, 67, 68, 172; 2.65 on 62— against 1.0 on the
  interiors. On map 2, played, **with no row drawn** (`+14` of every row
  zeroed, every page still running) **1.0**; with only the first 70 rows
  drawn, 1.0; with 80, 2.0. The frame does not fit because of drawing the
  events, not running them. Draft:
  [`desert-nightmare-the-towns-run-at-half-speed`](https://github.com/lautarodragan/the-advocate-work/blob/main/tasks/desert-nightmare-the-towns-run-at-half-speed.md).

**Why today's baking does not take them.** Two rules already move still
events into the background, and neither takes these:

* `decorative_events` ([Baked decorative events](../../../divergences/baked-decorative-events.md))
  wants a pose inside its 16x16 cell; a building piece is 24x32. Of the
  game's pieces, 436 qualify, and **434 of the 436** are refused because
  painting them "moved the background's CGRAM split" (65 maps in the
  log).
* `baked_events` ([Still events drawn into the background](../../../divergences/still-events-drawn-into-the-background.md)),
  off in this project. Built with it on, it draws **32 events into the
  background, on four maps** (48, 104, 105, 182), and keeps 73 more as
  objects on five because they "move its colours or its tiles past the
  layouts"; **no town map is a candidate** —map 2's panorama goes in mode
  1, which the rule does not take, and on maps 4 and 5 the log names no
  event either way; why, the build did not say and this study did not
  trace—.

**What it would gain.** Taking the 2,386 pieces with no commands, one map
of the 189 keeps more than 50 drawn events (172, with 58); taking every
one with a single look —their pages still run and block, only the drawing
moves—, no map keeps more than 16. That is under every limit above: the
cache, the table, the RAM's rows, the line and the frame.

*The console's*: 128 objects, 32 on a line and 34 slivers, and the VRAM
the poses take. *Ours*: carrying RM2k's "a building is events" into
objects. SNES games draw their towns into the background tilemaps, with
the priority bit for what goes over the characters, and keep objects for
whoever moves; a few rotate which objects go first each frame (the
priority rotation of `OAMADDH`) so that an overloaded line flickers
instead of losing the same ones.

### 6. Messages without a box

The game's message box is two pictures, `TextboxInnen` and
`TextboxRahmen`, shown by common 2 before every message over a
transparent window. As common pictures they fit on 14 of the 189 maps
("their small pictures fit on 14 of the 189 maps" in the log); on the
rest the text goes straight over the map —a message with a face gets the
System's frame, one without gets nothing— and the map's upper layer cuts
it. *Ours*. Already decided:
[`desert-nightmare-own-message-frame`](https://github.com/lautarodragan/the-advocate-work/blob/main/tasks/desert-nightmare-own-message-frame.md)
(the frame as the game's System, through the port's patches).

### 7. Smaller things seen

* **The scenes' pictures**: the dog dragged into the house (map 11,
  `Hund_Animation1..4`, placed by the dog's screen position) and 81 more
  `picture-not-shown` on 38 maps outside the menu. The scene plays; what
  it shows is missing.
* **Music**: all 19 songs are MP3 or WAV through Harmony; with MSU-1
  they play, without it the game is silent (0 seconds of SPC700 stream
  each in the log). The title's is
  [a task in the wishlist](https://github.com/lautarodragan/the-advocate-work/blob/main/tasks/desert-nightmare-title-is-silent-without-msu-1.md).
* **Space is not a limit**: `just space` leaves 17 whole banks and 18
  halves free, and 216 KB of text.

## The ranking

What would make it playable, cheapest first within each step; "unblocks"
is what the player gets.

    #  what                                      cost      gain / unblocks                                whose limit
    1  every event on the cell asked             small     day 1's box and the gate's middle: the        ours
       (event_in_cell, bump, step, twins)                  story past the alley; any stacked event
    2  ControlVars: the hero's map id            small     every item used (commons 20 and 14): the      ours
                                                           lamp, and every puzzle after it
    3  the number locks: a digits-only window,   small to  the end of day 1 (the safe), day 2's chest,   ours
       and their codes past 16 bits              moderate  day 3's door and chest
    4  the message box as the game's System      small to  every message readable, on every map          ours
       (own-message-frame, decided)              moderate
    5  the game's menu drawn                     large     choosing items, reading the notes with the    partly the
                                                           codes, the maps: without it the game is       console's
                                                           played blind
    6  the towns' pieces into the background     large     the buildings, the white holes on 34 maps,    the objects
       (24x32 pieces, the split allowed to                 half speed on 18, the cut lines on 59, the    are the
       move, pages kept)                                   RAM's trims                                   console's
    7  the scenes' pictures                      moderate  the dog's scene and ~80 more shows            ours
    8  music without MSU-1                       moderate  sound on a cartridge without MSU-1            ours

**1 to 3 are what makes it a game**: with them the story goes past day 1
—as far as the next thing nobody has seen, since day 2 was not reached—.
They are independent, each a few routines, and can go in parallel. 1 is
a runtime routine and needs a test over a stacked cell —ABL's suite or a
case of this game—; 2 is one field and a table of RM2k map ids; 3 is the
only one with a design question: sixteen bits are the ROM's choice, and a
lock only asks "is it equal", so comparing the low sixteen bits of what
was entered against the low sixteen of the code is one answer, and
32-bit variables for those two another.

**4 to 6 are what makes it look and feel like the original**, and 6 is
the big one: it is the white rectangles, the speed and the cut crowds at
once, on the maps where the game happens. Its first step is measuring
what `baked_events` and `decorative_events` leave out map by map with the
pieces allowed to spill —the 2,386 with no commands are a list already—,
and whether letting the split move costs a shade on the panoramas. 5
needs a decision before code: the pictures' path (a big picture with
small ones over it, research option 4) or a menu described as data that
replaces commons 4 to 22, as A Blurred Line's console did.

## How it was measured

* **The build**: `just cartridges` in `desert-nightmare/`, then `just
  culprits` and `just space`; `build/release/degradations.json` and
  `converter.log` for the counts and the per-map lines.
* **The playthrough**: a throwaway script over the ports' `games.py` and
  `play_script.py` —`ScriptedPlay`, its floor plan and its A taps— in
  stages, each saved and loaded with libretro's `retro_serialize`, so a
  stage could be replayed from the previous one. The ticks are the
  harness's; the frames per tick count frames until `ticks` changes.
* **The sweep**: every map entered with the harness's door teleport to
  its start cell on a new game, 240 ticks tapping A while a page runs and
  walking once there is control; the frames per tick over those ticks,
  `dbg_time_over` and `dbg_range_over` zeroed on arrival and read at the
  end. "No control" on most maps is a new game's first-visit scene, not
  a hang.
* **The CPU probe**: map 2 from the playthrough's state, 120 ticks
  standing, as is; with byte `+14` of every `npc_buf` row zeroed; and
  with only the first N rows left.
* **The events**: the dump's `data/maps/*.json`, counting the events
  that draw something, those whose pages all draw the same look with
  nobody moving them (`MoveEvent`, `SetEventLocation`,
  `SwapEventLocations`, `FlashSprite`, `ShowBattleAnimation`, or a
  branch on their facing, in any page or common event), and those with
  one page, no commands and no conditions.
* **RPG_RT**: EasyRPG Player, a new game started on each map at the same
  cell (`--start-map-id`, `--start-position`), for the towns; RPG_RT 2015
  under Wine for the menu. The port's captures are 256x224 and the
  original's 320x240: compared by eye.
* **Not measured**: day 2 onwards —nothing past the warehouse's lamp was
  played—; the chase sequences' monsters; bsnes beyond the boot test's
  own; the night maps' tint, which the teleport clears.
