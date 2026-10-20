# Events that do not fit in the RAM

## What

Every event of a map is a row of three tables in bank 0's RAM —its cell,
its flags and its active page—, and there are `EV_MAX` rows
(`rm2k_snes_mapdata::record::EV_MAX` in the-advocate). In several places of
the runtime an event's index is a byte, and the RAM is full: raising the cap
is not an option. In RM2k there is no such cap.

A map with more events than that used to stop the build. Now the converter
decides, before any stage reads the map, which events the console keeps
(`trim.rs` in `rm2k-snes-convert`), in two steps.

**First, the twins share a row, and nothing is lost.** Events whose pages
are the same, field by field, that draw nothing, never move and that nobody
names —with pages that run, or invisible walls— keep only the row of the
one with the lowest id. The others are a cell in a table in ROM, behind the
map's positions, which the runtime looks at when no row is on the cell the
hero steps on or faces (`twin_in_cell` in `runtime/action_key.s`): the cell
fires the first one's page, which is the same page. That is what most of
those maps have too many of: a strip of seventy invisible touch events
along an edge, each a `Teleport` to the same place; a grid of two hundred
invisible signs on the hero's layer, each the same message. What can not be
a twin, because it would be done on the first one and in RM2k it is its
own: a page that is autostart or parallel —it runs once per event—, that
erases its own event, or that names "this event".

A twin on the hero's layer stops him like the event it stands for: his
step into its cell is refused if the first one's active page is on his
layer (`hero_blocked_rest` in `runtime/twins_block.s`). An event that walks
into a twin's cell is not stopped, which RM2k would do: asking it for every
walker cost A Blurred Line's race, with forty cars, more frames than its
lap is allowed.

Until 2026-10-17 a twin could not be on the hero's layer, and the runtime
did not look at the twins when a step started. Zounds' maps 47 and 51 —grids
of info signs, the same page byte for byte— lost their signs by the hundred
and, ranked after them, the exits on 47's east edge, which the story goes
through.

**Then, if that is not enough, the rest is left out**, and an event left
out **stops existing** for the cartridge: the map is read without it by the
converter, the event compiler and Wally, so the indices agree; a page that
names it does not compile, as with any event that does not exist. The
converter says which ones, map by map, with what each one loses, in a
WARNING.

## Which ones go

From what loses the least to what loses the most: the ones that do
nothing —no command, nobody names them, they draw nothing and do not
block—; the still walls, which lose nothing (see below); the ones that are
only seen; the ones that are seen and are a still wall; invisible walls on
the hero's layer that can stop being one; the ones that are seen and block
that way; the ones with pages that run; the exits,
whose pages carry a `Teleport`; and last the ones something names
(`MoveEvent`, `SetEventLocation`, `FlashSprite`, a variable that reads
their cell, a `CallEvent` of theirs…), whose loss takes those pages along. Within each class, the highest id first —the
events placed last in the editor—: a rule anyone can redo by reading the
map. A first one of twins that goes takes its twins with it.

**A still wall left out stays a wall.** An event that runs nothing and
nobody names, with every page on the hero's layer, none that walks or has
a route of its own, and one page without conditions —so some page is
always active, whichever it is— blocks its cell forever, and nothing else
about it can be seen but its drawing, if it has one. Left out, its cell
goes closed in the map's passability, as Wally already did with a
decorative event on that layer: the hero is stopped where he was. Pages
under conditions are not read: if any of them is below or above the hero,
or none is free of conditions, it is a wall that can go away, and it is
ranked as a wall that is lost. The difference with RM2k is who else it
stops: an event below or above the hero that walks into that cell passes
in RM2k (`WouldCollide` in EasyRPG's `game_map.cpp`) and is stopped by the
passability here, as it is by a baked decoration.

**An exit left out is not silent.** If an exit has to go, one more event
goes, and the exit with the lowest id among those left out keeps its row
with its pages replaced by a box —"An exit left out here: the map has more
events than the console holds"—, as a `Teleport` to a map the cartridge
does not carry does; the other exits left out share that row as twins, so
each of their cells shows the box. It is one exit less than the cap
would allow, in exchange for the player seeing why the way is closed
instead of standing on a cell that does nothing. The box keeps that exit's
trigger and layer: a left-out exit fired another way than that one shows
nothing.

## Measured

On 2026-10-10 the cap stopped 38 maps of Vampires Dawn and 11 of Desert
Nightmare, from 121 events to 328. With the rule alone, 22 maps of Vampires
Dawn lost events with pages that run —most of them map exits—. With the
twins no map of either game loses a page: 24 maps of Vampires Dawn fit,
and the other 14 lose decoration only —two events of map 164 also block—;
2 maps of Desert Nightmare fit, and the other 9 lose decoration and, in 8
of them, invisible walls. A Blurred Line has no map over the cap.

On 2026-10-20, Desert Nightmare's maps 4, 14, 61 and 66 still lost
twenty-odd invisible walls each and as many drawn ones —counters, shelves,
the edges of rooms—, and Vampires Dawn's 164 two drawn ones; the hero
walked through them. Every one of them was still but four of map 14,
which a switch moves below the hero, and those are kept. Since then no
map of either game loses a wall: see
[the task](https://github.com/lautarodragan/the-advocate-work/blob/main/tasks/invisible-walls-lost-when-trimming-events.md).

On 2026-10-17, with twins on the hero's layer and invisible walls as twins,
Zounds' maps 47, 51 and 57 need 38, 26 and 16 rows and lose nothing; its map
178, a mini-game of 314 coins that are drawn, touch and erase themselves,
keeps 120 rows and its four exits, and leaves 206 coins out, which no
folding saves. No map of the eight games leaves an exit out. Folding the
copies that differ in one number of one command —a switch, a `Teleport`'s
destination row— was measured too: on Zounds' four maps it saves 10, 10, 3
and 0 rows more, and it is not done.

## How to reproduce it

`trim.rs` has the rule's tests; `events/tests/command_line.rs` and
`wally/tests/too_many_npcs.rs` check that a stage reads the map without
the events left out. A Blurred Line's suite has `test_twins.py`, on a
cartridge that tells the converter the RAM holds 16 events
(`--test-events-max`): the bridge of map 255 keeps two rows for its 61
creaking boards —one of them is named by a common event—, and stepping on
one that has no row still counts. The ports' `tests/test_invisible_walls.py` checks that neither Desert
Nightmare nor Vampires Dawn loses a wall, and walks into one of Desert
Nightmare's map 4 that is left out; `wally/tests/decorative.rs` checks
that the cells the converter passes are closed. The ports' `tests/test_zounds.py` walks
into an exit of Zounds' map 47 that the cap used to leave out, and, on a
cartridge that tells the converter the RAM holds 12 events, into one left
out, which shows the box.

## When it stops being enough

When what is left out is decoration that matters, or a wall that can go
away. The decoration has an answer that loses nothing and is not done: it
could be baked into the background, as `decorative_events` does (see
[Baked decorative events](baked-decorative-events.md)); but the cut
happens before Wally sees the map, so an event left out is not baked
either. A wall under a switch has none: it needs its row.

The still walls could also go to the passability **before** the cut, in
every map and not only in the ones over the cap: they would free rows for
everything else and cost the runtime nothing per frame. Not done: it
changes the cartridge of every game that has one, A Blurred Line's too,
for maps that fit, and it is the same divergence —who else the wall
stops— in maps that have no reason to pay it.

And when the twin table is what costs: it is walked whole, six bytes a
cell, on each step of the hero on a map that has twins. Zounds' map 47 has
246 twin cells; as runs of the same event along a row they would be 104,
and as rectangles 44 —map 51: 188, 37 and 20—. Not done: the cells fit,
and the walk has not been measured to cost a frame.
