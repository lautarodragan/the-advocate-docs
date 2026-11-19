# Can the towns be baked? (2026-11-19)

> **Measured on 2026-11-19** by Cardón, on Desert Nightmare's `release`
> built from the-advocate-ports `1ea90bf` with the tools of the-advocate
> `62b6e5cb`, and with a throwaway prototype of Wally on top of that
> commit (branch `cardon-dn-towns` in the-advocate, `cb45d5a4`, never to be
> merged).
> It is a snapshot: the numbers are from that day and this file is not
> updated. How each one was obtained is at the end, in "How it was
> measured". The task is
> [`desert-nightmare-can-the-towns-be-baked`](https://github.com/lautarodragan/the-advocate-work/blob/main/tasks/desert-nightmare-can-the-towns-be-baked.md).

[What it takes to be playable](2026-11-19-what-it-takes-to-be-playable.md)
ranked the towns sixth and the biggest: hundreds of 24x32 pieces of
buildings drawn as objects, which leave white holes, cut lines and half
the speed on the maps where the game happens. taro asked whether that
could be solved generically or only by changing the game heavily. This
is the answer, map by map.

## Summary

* **Yes, generically.** Desert Nightmare is a painted game: **161 of its
  189 maps have a panorama fixed to the map** —the size of the map, or one
  screen—, and the pieces stand on it. Painting every still piece that
  always shows one look **into that panorama**, with the ChipSet's few
  tiles folded in, makes each of those maps one picture, which the
  converter already knows how to carry: a [painted map](../../sunset-over-imdahl/research/2026-10-11-mode-1-for-the-painted-maps.md),
  as Sunset Over Imdahl's. What RM2k draws over a character —a roof, the
  top half of a wall the hero walks behind— goes in front of the hero
  with the same mechanism that already puts the map's pictures there.
* **On the console, with a prototype**, on the 28 towns: the **18 slow
  maps run at 1.0 to 1.13 frames per tick** (1.13 on map 12 and 1.07 on
  172, where a first-visit scene runs; 2.0 to 2.65 today), **only one of
  the 28 puts too many objects on a line** (27 did), and **no town keeps
  more than 16 drawn events**; the exception is map 35, the exit, whose
  27 people walk. The buildings are whole: the captures of that day are
  named in the task.
* **The 189 maps**: 133 have nothing measured wrong today; **48 fit**
  with the bake and some of today's rules relaxed; **8 are short**, by 1
  to 12 sprite slots or 2 slivers on a line. The table is at the end.
* **It is not the colours, nor the tiles.** A painted map uploads only the
  cells the screen shows, so VRAM is no limit; its seven palettes of
  fifteen keep the composite at a mean distance of 5.5 in 8-bit RGB
  (10.8 at worst), better than today's panoramas without the pieces
  (7.1). Of the ~3,000 pieces, 156 pixels in all go to the wrong side of
  the hero, on one map.
* **It is the cartridge's room.** A painted map carries every 8x8 cell of
  the map: painting all 135 maps the feature would take needs **86.5 banks**,
  and the cartridge has 126 for data, 91 of them taken. Only **29.5% of
  those cells are distinct** (25.5 banks): to paint every map, the cells need to be
  stored **once each**. **The 55 maps that need it fit as they are**: on
  one cartridge, 115 banks.
* **Three of today's rules have to change and two can be left alone.**
  The bake has to happen **before the RAM's 120-row trim**, which today
  drops up to 41 pieces a map before anything can bake them (the white
  roofs of maps 4, 14, 61 and 66); a `CallEvent` must not count as "can
  go away" when no common event erases (the pages call the message box,
  commons 2 and 3); and the hero's-layer pieces' top halves must be
  allowed to spill, in front. The CGRAM split and "the panorama shows
  through" stop mattering: the pieces never go into the background's
  cells.
* **What does not fit after that** is a handful of maps with many events
  that change look or walk —the cellars, the mine, the apartment at
  night, the exit's crowd—: short by 1 to 12 sprite slots. None needs the
  game changed for the towns; map 35 is the only one a patch would help.

## 1. What the pieces are

Every event of the dump with a page that draws something, 4,153 over the
189 maps, by what its pages do (a throwaway script over
`data/maps/*.json`, see the end):

    kind              events   what it is                                   how it can bake
    one page, still    2,429   one page, no commands, no conditions, no     plainly: a drawing
                               move type, nobody names it
    one look           500     every page draws the same CharSet, index,    plainly, keeping its pages:
                               pose and layer; nobody moves it              they run, block and trigger
    above the hero     589     a page on the layer above (485 one look)     in front of the hero
    pages              311     looks that change by switch —doors, lights,  the posed mechanism, a cell
                               day and night—; 119 have a page with none    per look; not over a painting
    animates           145     stepping or spinning in place                as an object
    moves              179     a move type, a route, `MoveEvent` or         as an object
                               `SetEventLocation` on it, a branch on its
                               facing

"Nobody names it" counts the map's pages and the common events. **3,414
of the 4,153 always show one look**: the first two rows, and 485 of the
third. Their shape: **2,786 are as wide as their cell
and reach up into the cell above** —most of them 16x32: the bottom half
of a wall and its top—, 1,052 stay inside their 16x16 cell, and the rest
are wider. The towns put them one
every other row, so each covers two rows of the building.

The number per map is in the table at the end; the towns carry 51 to 146
still pieces each.

## 2. Why each one is rejected today, and what each rule protects

Two rules move still events into the background today —[Baked
decorative events](../../../divergences/baked-decorative-events.md) and
[Still events drawn into the background](../../../divergences/still-events-drawn-into-the-background.md)—,
and one takes events out before either sees them, [Events that do not
fit in the RAM](../../../divergences/events-that-do-not-fit-in-the-ram.md).
With `[graphics] baked_events = true` (off in the project) and
`decorative_events = true` (on), the converter's log says why each
candidate stays an object. Summed over events, on the build of that day:

    events  maps  reason in the log                                 what it protects            here
    572     17    someone can stand behind what it draws past its   RM2k drawing that           real: the hero walks behind
                  cell                                              character under the piece   the buildings; relax: in front
    434     65    decorative event(s) moved the background's        the ground's colours        moot: a panorama baked into
                  CGRAM split                                                                   the background already merges
    194     25    it is drawn past its cell and can go away         a cell drawn once that      conservative: every page calls
                                                                    would stay drawn            commons 2 and 3; none erases
    109     17    the panorama shows through a cell it is drawn on  a cell with panorama in it  conservative for a panorama
                                                                    when the panorama moves     fixed to the map
    81      22    a page puts it above the hero                     BG1 at priority 0 is under  relax: in front
                                                                    the characters
    73      5     drawn into the background they move its colours   VRAM and the colours        real, on ChipSet maps
                  or its tiles past the layouts
     —      118   (no line) the posed bake does not run on a map    BG1 holding the panorama    moot over a painting
                  whose panorama goes baked (111) or mode 1 (7)
     —      7     (before Wally) the RAM's 120 rows leave events    RAM                         real; 150 of the 183 drawn
                  out                                                                           ones it leaves out are pieces

Rule by rule:

* **Drawn past its cell, someone behind it.** A 16x32 piece on the hero's
  layer covers the cell above its own, and on the towns that cell is
  often floor the hero walks on —behind the shop, along the motel's
  wall—: RM2k draws him under the piece there. In the background he would
  walk over it. The rule is right, and the way through is the front:
  those pixels go in front of the hero, as the map's pictures already do
  over a painted panorama (BG1's priority for a whole 8x8 cell, BG2's
  tiles for an edge). Of the 3,014 pieces the prototype painted, 2,315
  have pixels that go in front.
* **The CGRAM split.** `same_split` takes decoration only over a ground
  that merges no colour (`without.background.merges.is_empty()`), and
  Desert Nightmare's one-screen maps bake their panorama into the
  background with 150 to 260 colours, so it always merges: every
  decoration is refused, on every such map. It protects a clean ground
  from losing colours to a decoration —Vampires Dawn's river, which
  blended cost 1,557 pixels of its ground—; here the ground has lost
  them already. **Painting into the panorama makes it moot**: Adeleine
  reduces the whole painting at once, pieces included, and nothing is
  compared with a version without them.
* **Can go away.** `always_there` in `posed.rs` takes any `CallEvent` in
  the event's pages as one that may erase it. Desert Nightmare's pages
  call common 2 and 3 —the message box's pictures— 1,635 and 1,667
  times, and **no common event of the game has an `EraseEvent`**. The
  rule is conservative here; it can look at what the call reaches:
  common events, and through them other calls.
* **The panorama shows through.** A posed cell is drawn once with what is
  under it; over a panorama that moves against the map that would be
  wrong. A panorama fixed to the map never moves against it. Moot over a
  painting, and conservative anywhere the panorama is fixed.
* **Above the hero.** The posed cells are BG1 at priority 0. In a painting
  a piece above the hero goes in front whole: no rule is relaxed, the
  front mask takes it.
* **The ways posed does not run on.** It needs the map's background apart
  from the panorama; on a baked or mode-1 map there is none. Moot over a
  painting.
* **The RAM trim.** `trim.rs` in the converter leaves events out of maps
  with more than 120 before any stage sees them, and Wally never learns
  of them: on maps 2, 4, 14, 59, 61, 66 and 172, 183 drawn events, 150 of
  them pieces that only lose their looks. That is the white of
  [`desert-nightmare-roofs-left-out-show-white`](https://github.com/lautarodragan/the-advocate-work/blob/main/tasks/desert-nightmare-roofs-left-out-show-white.md):
  the prototype paints them from the whole map, and map 4's roofs come
  back. A piece with no commands baked into the painting needs no event
  at all, so baking before trimming also empties those maps under 120.
* **Past the layouts.** On the five ChipSet maps of the mines (48, 102,
  103, 105, 106), the posed blocks run out of background tiles. Real; it
  is today's mechanism, and those maps have no panorama to paint into.

## 3. Whether it fits: the prototype

The prototype —`rm2k-snes/wally/src/town.rs` on the branch, about three
hundred lines— forces the bake on a map: every drawn event that always
shows one look, nobody moves, flashes, turns or animates, whose first
page asks for nothing and that nothing erases, is painted into the
panorama in RM2k's order (layer, then row, column and id; a translucent
one blended), and **loses its row**: its pages still run, block and
trigger, as an event the table trims does. What RM2k would draw over a
character goes into the front mask: the whole of a piece above the hero,
and of a piece on the hero's layer the pixels past its cell that fall on
a cell someone can stand on, above it or to its left —Wally's
`standing`—. The ChipSet's layers are folded into the painting too, the
lower under the pieces and the upper over them and in front, so that the
map is one picture and goes painted. And the events the RAM's trim left
out are read from the whole map and painted the same way.

Built four ways: every map (the converter's log for all 189; the
cartridge does not link, see "The room"), the 28 towns with 30 or more
drawn events, the 27 other maps that put too many objects on a line
today, and those 55 together. The last three were played.

### Drawn events, slots and lines

Over all 189 maps the prototype paints **3,014 pieces**, 150 of them
ones the RAM's trim had left out. What stays an object is what changes
look (pages by switch), walks or animates: **at most 16 drawn events on
any town**, 31 on the mines' ChipSet maps 48 and 105, 27 on map 35.

On the 28 towns played, a door's teleport into each and 240 ticks:

    map                    today: frames/tick   prototype   lines over the PPU's 34 slivers
    2, 3, 4, 6, 11, 13,    2.0                  1.0         today 27 of the 28 (all but
    14, 43, 59, 61, 67, 68                                  138); with the bake, only 35
    5                      2.55                 1.0
    62                     2.65                 1.0
    12                     2.0                  1.13        (a first-visit scene runs)
    60                     2.0                  1.05
    64                     2.0                  1.0
    172                    2.0                  1.07        (a first-visit scene runs)
    35                     1.2 to 1.8           1.56        27 people who walk: still over

On the 27 other maps that put too many objects on a line today —the
interiors: the motel, the bar, the police station, the apartments, the
cellars— baked on a cartridge of their own: **only map 44 still goes
over**, by 2 slivers (section 4).

The cache's slots, the converter's count by where the events start:
today 35 maps have more events on screen than slots or trim events (80
on screen for 15 slots on map 2, 90 for 23 on 5, 74 for 11 on 172); with
the bake, 8 —see section 4—. Of the other towns, 10, 15, 63, 65, 66,
72 and 121 ran at 1.2 to 1.8 frames per tick today and at 1.0 baked.

### The colours

A painted map is mode 1, BG1 at 4 bits, seven palettes of fifteen; each
8x8 cell takes the first palette that has all its colours, and Adeleine
chooses the palettes for the whole painting. The towns' composites have
300 to 400 colours (map 2: 395; 172: 816). Measured as the mean distance
between the composite and what the painting decided, in 8-bit RGB, over
the 38 painted maps without map pictures drawn into them (the night maps
blend a picture over the painting, which this cannot separate): **5.5 on
average, 10.8 at worst** (map 10; one step of the console's 5-bit
channels is 8). Today's panoramas alone, without the pieces, on the 24
of those maps that had one in a layer of its own: 7.1. Map 2 today keeps
12 of its panorama's 240 colours in mode 1; painted with its pieces, 72
of 395.

### The tiles

A painted map holds in VRAM only the 990 cells the screen can show, and
uploads a column or a row as the camera moves (`runtime/painted.s` in
the-advocate): **any number of distinct tiles fits**. The painted layout
gives sprites 256 tiles, against 384 or 512 in the others, which is
why a few one-screen maps that bake into the background today lose
slots if they go painted (section 4). What goes in front of the hero by an edge takes
BG2 tiles, 127 of them besides the blank one: 156 pixels go to the wrong
side of the hero on map 172 for want of more, 0 on the other 134.

The blocks and the grid shrink: map 2's ChipSet layers took 65 16x16
blocks, folded into the painting 19 (only the passability is left).

### The room

A painted map carries **every** 8x8 cell, 32 bytes each, column by
column, so that a column goes up in one DMA. The 135 maps that go
painted with every rule relaxed take **177,156 cells, 86.5 banks**; the
cartridge has 126 banks for data and the release takes 91 today, so it
does not link ("the 126 banks the cartridge has for data are taken").
**52,187 of those cells are distinct within their map (29.5%), 25.5
banks**, plus 5.4 for an index of two bytes per cell; across maps,
49,985. The 28 towns alone, carried whole, take the release to 100
banks; the 55 maps that put too many objects on a line today, 115, and
the cartridge links. On that cartridge only maps 35 and 44 go over a
line.

## 4. What does not fit, and by how much

With every rule relaxed, eight maps still have more events on screen
than sprite slots, by the converter's count —one of them fine today—,
and one more still puts too many objects on a line when played:

    map  name                 on screen / slots   what stays an object               what would fit it
    35   Ausgang              27 / 15  (−12)      27 people who walk out             a patch: fewer walkers, or
                                                                                     the rotation of OAM priority
    71   Apartment 4 Nacht    17 / 11  (−6)       12 that change look, 2 walk        posed over a painting
    18   Apartment 4          15 / 11  (−4)       14 that change look                keep it baked (fine today)
    137  Werkstatt-Keller     15 / 11  (−4)       12 that change look                posed over a painting
    33   Apartment 4 Nacht    14 / 11  (−3)       12 that change look                posed over a painting
    105  Mine 4               25 / 23  (−2)       24 that change look (ChipSet map)  posed, as today, with room
    138  Kellergewölbe 1      16 / 15  (−1)       12 that change look                posed over a painting
    172  Alte Minensiedlung   12 / 11  (−1)       12 that change look                posed over a painting
    44   Post-Innen           36 slivers / 34     8 drawn with a ChipSet tile        paint the tile events too

Map 18 is fine today and is in the list only because going painted
halves the sprite VRAM: it should keep its way. 33 and 71, the same
apartment at night, put nine characters on a line today and need the
front, so they have to go painted, and lose the slots. Map 44 keeps eight
events drawn with a ChipSet tile, which the prototype leaves as objects;
over a painting they could be painted as the pieces are. The rest are
events that **change look by switch** —lights on and off, doors that
open—, which the posed mechanism takes on a background of blocks and
cannot take on a painting, whose BG1 is a ring of uploaded cells. Map
35 is the exit's crowd, which walks: no bake helps, and a patch like A
Blurred Line's map 13 fork —fewer walkers on the line, or drawn in
turns— is the only way.

One white hole is left in the prototype, on map 61, the workshop at
night: five events that change look, which the RAM's trim leaves out and
nothing draws. With the pieces baked before the trim, the plain ones stop
being events and those five keep their rows (the map has 192 events, 145
of them still pieces).

The ROM: every map painted cell by cell does not fit (section 3); the
55 that need it do, with 11 banks to spare. With the cells stored once,
every map does.

## The recommendation

**A converter feature, not patches and not a redesign**: still events
painted into a panorama fixed to the map. In order, with what each part
costs:

1. **The pieces into the painting** (Wally, moderate: the prototype's
   selection and painting are ~300 lines, plus its wiring). Behind a
   project knob, as `painted_maps` is —or as part of it—: on a map whose
   panorama is fixed to it, every still event with one look always is
   painted into the panorama with the ChipSet's static layers, in RM2k's
   order, and what RM2k draws over a character goes into the front mask
   that the map's pictures already fill. An event with no commands stops
   being an event, as a decorative one does; one with pages keeps them
   and loses its row. A map goes painted only if that leaves its drawn
   events within its slots —or keeps the way it has today otherwise, as
   map 18 of section 4 should—, and baked if it is one screen and nothing
   goes in front. **The `CallEvent` rule** goes with it: a call
   can erase only if what it reaches erases (draft
   [`a-call-counts-as-erasing-a-still-event`](https://github.com/lautarodragan/the-advocate-work/blob/main/tasks/a-call-counts-as-erasing-a-still-event.md)).
2. **Before the RAM's trim** (the converter, small to moderate): the
   pieces are known before `trim.rs` counts the 120, so a map like 4
   keeps its roofs and the pieces with no commands leave the table
   instead. This is what [`desert-nightmare-roofs-left-out-show-white`](https://github.com/lautarodragan/the-advocate-work/blob/main/tasks/desert-nightmare-roofs-left-out-show-white.md)
   asked for, and it holds for `decorative_events` as it is too.
3. **A painted map's cells stored once** (the converter and
   `runtime/painted.s`, moderate): a column is uploaded today with one
   DMA from the cartridge; with the cells deduplicated it needs the
   column's 30 cells gathered by an index —30 small DMAs or a copy into a
   WRAM buffer and one DMA—, which takes vblank time that was not
   measured. Not needed for the 55 maps that need the bake (115
   banks); needed to paint every map: 86.5 banks become ~31 (draft
   [`painted-maps-carry-every-cell`](https://github.com/lautarodragan/the-advocate-work/blob/main/tasks/painted-maps-carry-every-cell.md)).

The events that change look by switch (311 in the game, at most 12 on a
map) can stay objects: with the pieces gone none of the towns is short of
slots. Posed over a painting —a ring cell rewritten when the page
changes— would take them, and is not needed for this game.

**Patches**: only map 35, whose crowd walks. **No redesign of the
game.**

### What it does to the other ports

* **A Blurred Line**: 12 of its 328 maps have a panorama fixed to the
  map, with 10 still one-look events on them; it does not use painted
  maps. Behind the project's knob, nothing changes; the move before the
  trim changes nothing on maps under 120 events. The `CallEvent` rule, if
  it also goes into `posed.rs`'s `always_there`, changes which events ABL
  bakes where a page calls a common event: its goldens would move, and
  that was not measured.
* **Zounds**: 6 maps with a fixed panorama and no drawn events on them:
  nothing, but for the same `CallEvent` caveat on its posed maps.
* **Vampires Dawn**: 18 maps with a fixed panorama, 27 still events on
  them (18 above the hero); one, map 257, has 12. Off unless asked.
* **Sunset Over Imdahl**, which is the other painted game: 63 of its 65
  maps have a fixed panorama, with 66 still one-look events on them (70
  drawn on map 58), and its painted maps would take them; the cells
  stored once would make its 60 painted maps smaller.

## The 189 maps

Columns: the drawn events (CharSet or tile pages); the still pieces that
always show one look (one page still + one look + above the hero, from
section 1); what is wrong today, measured; the way the panorama goes
today and with every rule relaxed and `painted_maps` on (a map with no
pieces may go painted for that knob alone); the pieces painted and the drawn
events left as objects; the rules a map needs relaxed (*front*: pieces
that draw over a character; *split*, *go-away*, *holes*, *above*: the
log's reasons of section 2 for that map; *way*: posed does not run on its
panorama's way; *trim*: the RAM trim leaves pieces out); and the
verdict:

* **fine today**: nothing measured wrong, no bake needed (the prototype
  still paints its pieces if asked);
* **fits, X relaxed**: fixed by the bake with those rules;
* **short by N**: still N events on screen past the sprite slots, by the
  converter's count, or what the line count says.

Today's problems: *slots a/b* the events on screen and the cache's
slots, *trim* the table's 104 or the RAM's 120, *line* over the PPU's 34
slivers (the 2026-11-17 count), *f/t* frames per tick (Hurón's sweep).
After: the same on the prototype; *line* and *f/t* only on the 55 maps
played.

| map | name | drawn | still pieces | today | panorama | painted / left | played | verdict |
|---|---|---|---|---|---|---|---|---|
| 1 | Init | 0 | 0 | — | mode3 → painted | 0 / 0 | — | fine today |
| 2 | Dusty Creek | 130 | 119 | slots 80/15, trim, line, 2 f/t | mode1 → painted | 99 / 16 | no line, 1.01 f/t | fits, front, split, way, trim relaxed |
| 3 | Bar | 81 | 81 | slots 80/31, line, 2 f/t | baked → painted | 81 / 0 | no line, 1.01 f/t | fits, front, split, way relaxed |
| 4 | Werkstatt | 154 | 146 | slots 79/23, trim, line, 2 f/t | flattened → painted | 110 / 4 | no line, 1.00 f/t | fits, front, go-away, above, trim relaxed |
| 5 | Wohngegend | 114 | 110 | slots 90/23, trim, line, 2.5 f/t | flattened → painted | 110 / 4 | no line, 1.00 f/t | fits, front, split, go-away, holes, above relaxed |
| 6 | Tankstelle | 79 | 76 | slots 77/23, line, 2 f/t | flattened → painted | 76 / 3 | no line, 1.00 f/t | fits, front, split, go-away, holes, above relaxed |
| 7 | WC | 1 | 1 | — | — → — | 0 / 1 | — | fine today |
| 8 | Tankstelle Innen | 0 | 0 | — | — → — | 0 / 0 | — | fine today |
| 9 | Strasse | 1 | 0 | — | mode3 → mode3 | 0 / 1 | — | fine today |
| 10 | Kleiner Garten | 52 | 51 | slots 37/15, line | baked → painted | 51 / 1 | no line, 1.00 f/t | fits, front, split, way relaxed |
| 11 | Kleine Gasse 1 | 54 | 53 | slots 53/23, line, 2 f/t | flattened → painted | 52 / 2 | no line, 1.00 f/t | fits, front, split relaxed |
| 12 | Kleine Gasse 2 | 70 | 65 | slots 58/27, line, 2 f/t | baked → painted | 63 / 7 | no line, 1.13 f/t | fits, front, split, way relaxed |
| 13 | Kleine Gasse 3 | 83 | 83 | slots 69/27, line, 2 f/t | baked → painted | 83 / 0 | no line, 1.00 f/t | fits, front, split, way relaxed |
| 14 | Polizeirevier | 150 | 142 | slots 80/23, trim, line, 2 f/t | flattened → painted | 107 / 9 | no line, 1.00 f/t | fits, front, split, go-away, holes, above, trim relaxed |
| 15 | Kleine Gasse | 81 | 77 | slots 79/31, line | baked → painted | 76 / 5 | no line, 1.00 f/t | fits, front, split, way relaxed |
| 16 | Motel - Erdgeschoss | 23 | 18 | line | flattened → painted | 16 / 7 | no line, 1.00 f/t | fits, front, split, go-away, holes, above relaxed |
| 17 | Motel - 1.Stock | 13 | 7 | line | flattened → painted | 6 / 7 | no line | fits, front, split, go-away, holes, above relaxed |
| 18 | Apartment 4 | 20 | 14 | — | baked → painted | 5 / 15 | — | fine today |
| 19 | Bar Innen | 25 | 23 | slots 21/19, line | flattened → painted | 17 / 8 | no line, 1.00 f/t | fits, front, split relaxed |
| 20 | Allzweckladen | 23 | 18 | line | baked → painted | 18 / 5 | no line, 1.00 f/t | fits, front, split, way relaxed |
| 21 | Lagerhaus | 6 | 3 | — | flattened → painted | 3 / 3 | — | fine today |
| 22 | Gang | 10 | 5 | — | flattened → painted | 4 / 6 | — | fine today |
| 23 | Raum 1 | 7 | 5 | — | baked → painted | 5 / 2 | — | fine today |
| 24 | Raum 2 | 11 | 8 | — | baked → painted | 6 / 5 | — | fine today |
| 25 | Raum 3 | 9 | 7 | — | baked → painted | 4 / 5 | — | fine today |
| 26 | Raum 4 | 9 | 5 | — | baked → painted | 5 / 4 | — | fine today |
| 27 | Raum 5 | 15 | 11 | — | baked → painted | 3 / 12 | — | fine today |
| 28 | Raum 6 | 11 | 3 | — | flattened → painted | 1 / 10 | — | fine today |
| 29 | Todesszenen | 0 | 0 | — | — → — | 0 / 0 | — | fine today |
| 30 | Lagerhaus | 1 | 0 | — | baked → baked | 0 / 1 | — | fine today |
| 31 | Bar Nacht | 28 | 24 | slots 25/23, line | flattened → painted | 19 / 9 | no line, 1.00 f/t | fits, front, split relaxed |
| 32 | Motel - 1.Stock Nacht | 14 | 10 | line | flattened → painted | 8 / 6 | no line | fits, front, split, go-away, holes, above relaxed |
| 33 | Apartment 4 Nacht | 18 | 12 | line | baked → painted | 5 / 13 | no line | short by 3 slots |
| 34 | Apartment 4 Traum | 7 | 6 | — | flattened → painted | 6 / 1 | — | fine today |
| 35 | Ausgang | 37 | 10 | slots 33/31, line | baked → painted | 10 / 27 | line, 1.56 f/t | short by 12 slots, line, 1.6 f/t |
| 36 | Wohnung Nr.7 | 21 | 16 | line | baked → painted | 9 / 12 | no line | fits, front, split, way relaxed |
| 37 | Wohnung Nr. 9 | 16 | 12 | — | baked → painted | 12 / 4 | — | fine today |
| 38 | Wohnung Nr.10-1 | 9 | 7 | — | baked → painted | 5 / 4 | — | fine today |
| 39 | Wohnung Nr.10-2 | 14 | 8 | — | baked → painted | 4 / 10 | — | fine today |
| 40 | Apartment 1 | 4 | 2 | — | baked → painted | 2 / 2 | — | fine today |
| 41 | Hinterhof | 53 | 51 | slots 44/27, line | baked → painted | 51 / 2 | no line | fits, front, split, way relaxed |
| 42 | Innen | 14 | 10 | — | baked → painted | 8 / 6 | — | fine today |
| 43 | Post | 99 | 93 | slots 85/15, line, 2 f/t | mode1 → painted | 93 / 6 | no line, 1.00 f/t | fits, front, split, way relaxed |
| 44 | Post-Innen | 14 | 11 | line | baked → painted | 3 / 11 | line | short by line |
| 45 | Kleines Haus-Innen | 3 | 0 | — | baked → baked | 0 / 3 | — | fine today |
| 46 | Hinterhof | 1 | 0 | — | baked → baked | 0 / 1 | — | fine today |
| 47 | Untergrund | 21 | 13 | line | — → — | 5 / 16 | no line | fits, split relaxed |
| 48 | Untergrund 2 | 62 | 47 | slots 15/11, line | — → — | 31 / 31 | no line, 1.12 f/t | fits, front, split, go-away, above relaxed |
| 49 | Untergrund 3 | 6 | 6 | — | — → — | 4 / 2 | — | fine today |
| 50 | Untergrund 4 | 4 | 3 | — | — → — | 0 / 4 | — | fine today |
| 51 | Lager | 16 | 12 | — | baked → painted | 7 / 9 | — | fine today |
| 52 | Unterkunft | 12 | 9 | — | baked → painted | 7 / 5 | — | fine today |
| 53 | Untergrund 1 | 2 | 1 | — | baked → painted | 1 / 1 | — | fine today |
| 54 | Untergrund 2 | 1 | 0 | — | baked → baked | 0 / 1 | — | fine today |
| 55 | Untergrund 3 | 1 | 0 | — | baked → baked | 0 / 1 | — | fine today |
| 56 | Untergrund 4 | 1 | 0 | — | baked → baked | 0 / 1 | — | fine today |
| 57 | Apartment 4 2.Nacht | 19 | 11 | — | baked → painted | 6 / 13 | — | fine today |
| 58 | Traum 2 | 16 | 6 | — | flattened → painted | 6 / 10 | — | fine today |
| 59 | Dusty Creek-Nacht | 125 | 121 | slots 79/15, trim, line, 2 f/t | mode1 → painted | 103 / 11 | no line, 1.00 f/t | fits, front, split, way, trim relaxed |
| 60 | Bar-Nacht | 81 | 73 | slots 54/23, line, 2 f/t | flattened → painted | 73 / 8 | no line, 1.05 f/t | fits, front relaxed |
| 61 | Werkstatt-Nacht | 152 | 145 | slots 80/23, trim, line, 2 f/t | flattened → painted | 108 / 3 | no line, 1.00 f/t | fits, front, go-away, holes, above, trim relaxed |
| 62 | Wohngegend-Nacht | 114 | 112 | slots 90/23, trim, line, 2.6 f/t | flattened → painted | 110 / 4 | no line, 1.00 f/t | fits, front, split, go-away, holes, above relaxed |
| 63 | Kleine Gasse 1 | 53 | 53 | slots 52/23, line | flattened → painted | 52 / 1 | no line, 1.00 f/t | fits, front, split relaxed |
| 64 | Kleine Gasse 2 | 69 | 65 | slots 57/23, line, 2 f/t | flattened → painted | 63 / 6 | no line, 1.00 f/t | fits, front, split relaxed |
| 65 | Kleine Gasse 3 | 83 | 83 | slots 69/23, line | flattened → painted | 83 / 0 | no line, 1.00 f/t | fits, front, split relaxed |
| 66 | Polizeirevier-Nacht | 146 | 146 | slots 82/23, trim, line | flattened → painted | 115 / 0 | no line, 1.00 f/t | fits, front, split, go-away, holes, above, trim relaxed |
| 67 | Kleine Gasse | 80 | 80 | slots 78/23, line, 2 f/t | flattened → painted | 79 / 1 | no line, 1.00 f/t | fits, front, split relaxed |
| 68 | Post-Nacht | 96 | 96 | slots 82/15, line, 2 f/t | mode1 → painted | 96 / 0 | no line, 1.00 f/t | fits, front, split, way relaxed |
| 69 | Motel - Erdgeschoss-N | 21 | 17 | line | flattened → painted | 15 / 6 | no line | fits, front, split, go-away, holes, above relaxed |
| 70 | Motel - 1.Stock Nacht | 13 | 9 | line | flattened → painted | 8 / 5 | no line | fits, front, split, go-away, holes, above relaxed |
| 71 | Apartment 4 Nacht | 20 | 12 | line | baked → painted | 5 / 15 | no line | short by 6 slots |
| 72 | Hinterhof | 52 | 52 | slots 43/15, line | mode1 → painted | 52 / 0 | no line, 1.00 f/t | fits, front, split, way relaxed |
| 73 | Apartment 1 | 4 | 2 | — | baked → painted | 2 / 2 | — | fine today |
| 74 | Allzweckladen-Nacht | 22 | 19 | line | baked → painted | 19 / 3 | no line | fits, front, split, way relaxed |
| 75 | Wohnung Nr.7 | 20 | 19 | line | baked → painted | 13 / 7 | no line | fits, front, split, way relaxed |
| 76 | Wohnung Nr. 9 | 16 | 12 | — | baked → painted | 12 / 4 | — | fine today |
| 77 | Wohnung Nr.10-1 | 9 | 7 | — | baked → painted | 5 / 4 | — | fine today |
| 78 | Wohnung Nr.10-2 | 14 | 12 | — | baked → painted | 8 / 6 | — | fine today |
| 79 | Post-Innen | 13 | 13 | line | baked → painted | 5 / 8 | no line | fits, front, way relaxed |
| 80 | Kleines Haus-Innen | 1 | 0 | — | baked → baked | 0 / 1 | — | fine today |
| 81 | Innen | 13 | 11 | — | baked → painted | 9 / 4 | — | fine today |
| 82 | Apartment 2 | 8 | 5 | — | flattened → painted | 1 / 7 | — | fine today |
| 83 | Apartment 2 | 8 | 5 | — | baked → painted | 1 / 7 | — | fine today |
| 84 | Hundehaus 1 | 6 | 5 | — | baked → painted | 4 / 2 | — | fine today |
| 85 | Hundehaus 2 | 6 | 3 | — | baked → painted | 2 / 4 | — | fine today |
| 86 | Hundehaus 3 | 15 | 7 | — | baked → painted | 2 / 13 | — | fine today |
| 87 | Hundehaus 1 | 6 | 5 | — | baked → painted | 4 / 2 | — | fine today |
| 88 | Hundehaus 2 | 5 | 4 | — | baked → painted | 4 / 1 | — | fine today |
| 89 | Hundehaus 3 | 10 | 10 | — | baked → painted | 9 / 1 | — | fine today |
| 90 | Hundehaus 1 | 1 | 0 | — | baked → baked | 0 / 1 | — | fine today |
| 91 | Hundehaus 2 | 2 | 1 | — | baked → baked | 1 / 1 | — | fine today |
| 92 | Polizeirevier-Innen | 26 | 21 | line | baked → painted | 17 / 9 | no line, 1.00 f/t | fits, front, split, way relaxed |
| 93 | Zellen | 24 | 20 | line | baked → painted | 15 / 9 | no line | fits, front, way relaxed |
| 94 | Verhöhrzimmer | 6 | 5 | — | baked → painted | 2 / 4 | — | fine today |
| 95 | Traum 3 | 10 | 4 | — | flattened → painted | 4 / 6 | — | fine today |
| 96 | Polizeirevier-Innen | 26 | 24 | line | baked → painted | 22 / 4 | no line, 1.00 f/t | fits, front, split, way relaxed |
| 97 | Zellen | 22 | 21 | line | baked → painted | 19 / 3 | no line | fits, front, way relaxed |
| 98 | Verhöhrzimmer | 4 | 4 | — | baked → painted | 3 / 1 | — | fine today |
| 99 | Schuppen | 8 | 6 | — | baked → painted | 6 / 2 | — | fine today |
| 100 | Allzweckladen | 2 | 1 | — | baked → painted | 1 / 1 | — | fine today |
| 101 | Tunnel | 2 | 2 | — | baked → baked | 0 / 2 | — | fine today |
| 102 | Mine | 34 | 32 | slots 30/27, line | — → — | 30 / 4 | no line, 1.00 f/t | fits, split, go-away, above relaxed |
| 103 | Mine 2 | 28 | 26 | — | — → — | 21 / 7 | — | fine today |
| 104 | Mine 3 | 3 | 2 | — | — → — | 0 / 3 | — | fine today |
| 105 | Mine 4 | 37 | 29 | slots 28/23 | — → — | 6 / 31 | — | short by 2 slots |
| 106 | Mine 5 | 17 | 10 | — | — → — | 2 / 15 | — | fine today |
| 107 | Mine 6 | 7 | 4 | — | — → — | 0 / 7 | — | fine today |
| 108 | Mine 7 | 6 | 3 | — | — → — | 0 / 6 | — | fine today |
| 109 | Mine 8 | 2 | 1 | — | — → — | 1 / 1 | — | fine today |
| 110 | Flucht | 1 | 0 | — | — → — | 0 / 1 | — | fine today |
| 111 | Mine 9 | 8 | 2 | — | — → — | 1 / 7 | — | fine today |
| 112 | Mine 10 | 10 | 9 | — | — → — | 9 / 1 | — | fine today |
| 113 | Unterkunft | 18 | 10 | — | baked → painted | 5 / 13 | — | fine today |
| 114 | WC | 1 | 1 | — | baked → painted | 1 / 0 | — | fine today |
| 115 | Büro | 11 | 8 | — | flattened → painted | 5 / 6 | — | fine today |
| 116 | Mine 1 | 1 | 0 | — | baked → baked | 0 / 1 | — | fine today |
| 117 | Mine 2 | 3 | 1 | — | baked → painted | 1 / 2 | — | fine today |
| 118 | Ausgang | 13 | 0 | — | baked → baked | 0 / 13 | — | fine today |
| 119 | Traum 4 | 12 | 6 | — | baked → painted | 4 / 8 | — | fine today |
| 120 | Apartment 6 | 7 | 7 | — | baked → painted | 6 / 1 | — | fine today |
| 121 | Kleiner Garten | 52 | 51 | slots 37/15, line | mode1 → painted | 51 / 1 | no line, 1.00 f/t | fits, front, split, way relaxed |
| 122 | Schuppen | 8 | 6 | — | baked → painted | 6 / 2 | — | fine today |
| 123 | Schneiderei | 5 | 5 | — | baked → painted | 5 / 0 | — | fine today |
| 124 | Hinterzimmer | 8 | 5 | — | baked → painted | 4 / 4 | — | fine today |
| 125 | Fleischerei | 14 | 8 | — | baked → painted | 2 / 12 | — | fine today |
| 126 | Apartment 3 | 3 | 1 | — | baked → baked | 0 / 3 | — | fine today |
| 127 | Apartment 5 | 5 | 4 | — | baked → painted | 3 / 2 | — | fine today |
| 128 | Fleischerei 2 | 10 | 6 | — | baked → painted | 2 / 8 | — | fine today |
| 129 | Fleischerei 3 | 4 | 1 | — | baked → painted | 1 / 3 | — | fine today |
| 130 | Lagerraum | 9 | 7 | — | baked → painted | 7 / 2 | — | fine today |
| 131 | Bar Nacht | 23 | 23 | line | flattened → painted | 18 / 5 | no line | fits, front, split relaxed |
| 132 | WC | 6 | 3 | — | baked → baked | 0 / 6 | — | fine today |
| 133 | Keller | 4 | 3 | — | baked → painted | 3 / 1 | — | fine today |
| 134 | Bar WC | 2 | 0 | — | baked → baked | 0 / 2 | — | fine today |
| 135 | Kleiner Raum | 1 | 0 | — | baked → baked | 0 / 1 | — | fine today |
| 136 | Werkstatt | 27 | 17 | slots 27/23, line | flattened → painted | 15 / 12 | no line | fits, front, go-away, above relaxed |
| 137 | Werkstatt-Keller | 27 | 16 | slots 28/19, line | flattened → painted | 14 / 13 | no line | short by 4 slots |
| 138 | Kellergewölbe 1 | 31 | 14 | slots 24/23 | flattened → painted | 12 / 19 | no line | short by 1 slots |
| 139 | Raum 2 | 15 | 11 | — | baked → painted | 10 / 5 | — | fine today |
| 140 | Raum 3 | 6 | 4 | — | baked → painted | 4 / 2 | — | fine today |
| 141 | Raum 4 | 9 | 8 | — | baked → painted | 8 / 1 | — | fine today |
| 142 | Raum 5 | 9 | 6 | — | baked → painted | 5 / 4 | — | fine today |
| 143 | Raum 6 | 21 | 10 | — | flattened → painted | 8 / 13 | — | fine today |
| 144 | Raum 7 | 11 | 5 | — | flattened → painted | 4 / 7 | — | fine today |
| 145 | Raum 8 | 18 | 13 | — | baked → painted | 8 / 10 | — | fine today |
| 146 | Raum 9 | 15 | 5 | — | baked → painted | 3 / 12 | — | fine today |
| 147 | Keller1_1 | 2 | 0 | — | baked → baked | 0 / 2 | — | fine today |
| 148 | Keller1_2 | 2 | 1 | — | baked → baked | 0 / 2 | — | fine today |
| 149 | Keller1_3 | 0 | 0 | — | baked → baked | 0 / 0 | — | fine today |
| 150 | Kellergewölbe 2 | 5 | 3 | — | baked → painted | 3 / 2 | — | fine today |
| 151 | Raum 2 | 2 | 2 | — | baked → painted | 2 / 0 | — | fine today |
| 152 | Raum 3 | 4 | 2 | — | baked → painted | 1 / 3 | — | fine today |
| 153 | Raum 4 | 21 | 11 | line | baked → painted | 9 / 12 | no line | fits, front, way relaxed |
| 154 | Raum 5 | 5 | 5 | — | baked → painted | 5 / 0 | — | fine today |
| 155 | Raum 6 | 8 | 6 | — | baked → painted | 6 / 2 | — | fine today |
| 156 | Raum 7 | 10 | 5 | — | flattened → painted | 5 / 5 | — | fine today |
| 157 | Raum 8 | 15 | 11 | — | flattened → painted | 9 / 6 | — | fine today |
| 158 | Raum 9 | 7 | 6 | — | baked → painted | 6 / 1 | — | fine today |
| 159 | Raum 10 | 9 | 8 | — | baked → painted | 8 / 1 | — | fine today |
| 160 | Raum 11 | 9 | 6 | — | baked → painted | 6 / 3 | — | fine today |
| 161 | Raum 12 | 11 | 5 | — | flattened → painted | 5 / 6 | — | fine today |
| 162 | Raum 13 | 4 | 2 | — | baked → painted | 2 / 2 | — | fine today |
| 163 | Keller2_1 | 2 | 0 | — | baked → baked | 0 / 2 | — | fine today |
| 164 | Keller2_2 | 4 | 2 | — | baked → baked | 2 / 2 | — | fine today |
| 165 | Keller2_3 | 2 | 0 | — | baked → baked | 0 / 2 | — | fine today |
| 166 | Keller2_4 | 2 | 1 | — | baked → painted | 1 / 1 | — | fine today |
| 167 | Auf Straße | 5 | 3 | — | — → — | 3 / 2 | — | fine today |
| 168 | Indianer | 12 | 11 | — | — → — | 11 / 1 | — | fine today |
| 169 | Im Zelt | 7 | 7 | — | baked → painted | 3 / 4 | — | fine today |
| 170 | Haus vor Siedlung | 13 | 6 | — | baked → painted | 6 / 7 | — | fine today |
| 171 | Vor dem Haus | 22 | 20 | line | flattened → painted | 20 / 2 | no line | fits, front, split relaxed |
| 172 | Alte Minensiedlung | 125 | 102 | slots 74/11, trim, line, 2 f/t | mode1 → painted | 98 / 16 | no line, 1.07 f/t | short by 1 slots, 156 px |
| 173 | Haus 1 | 5 | 3 | — | baked → painted | 3 / 2 | — | fine today |
| 174 | Haus 2 | 12 | 7 | — | baked → painted | 4 / 8 | — | fine today |
| 175 | Haus 3 | 12 | 10 | — | hidden → painted | 6 / 6 | — | fine today |
| 176 | Raum 2 | 3 | 1 | — | baked → painted | 1 / 2 | — | fine today |
| 177 | Mine | 11 | 7 | — | — → — | 3 / 8 | — | fine today |
| 178 | Raum 2 | 8 | 7 | — | — → — | 7 / 1 | — | fine today |
| 179 | Raum 3 | 9 | 7 | — | — → — | 7 / 2 | — | fine today |
| 180 | Raum 4 | 5 | 3 | — | — → — | 2 / 3 | — | fine today |
| 181 | Raum 5 | 12 | 12 | — | — → — | 12 / 0 | — | fine today |
| 182 | Raum 6 | 22 | 11 | — | — → — | 4 / 18 | — | fine today |
| 183 | Alte Mine | 4 | 2 | — | baked → baked | 0 / 4 | — | fine today |
| 184 | Auf Straße 2 | 14 | 13 | — | baked → painted | 13 / 1 | — | fine today |
| 185 | Credits | 0 | 0 | — | — → — | 0 / 0 | — | fine today |
| 186 | Keller2_5 | 1 | 0 | — | baked → baked | 0 / 1 | — | fine today |
| 187 | Abspann 1 | 0 | 0 | — | mode3 → mode3 | 0 / 0 | — | fine today |
| 188 | Abspann 2 | 4 | 4 | — | flattened → painted | 1 / 3 | — | fine today |
| 189 | Abspann 3 | 1 | 0 | — | flattened → flattened | 0 / 1 | — | fine today |

## How it was measured

* **The builds**: `just cartridges` in `desert-nightmare/`, with
  `[graphics] baked_events = true` for today's reasons (the project has
  it off; `decorative_events` is on), and with `painted_maps = true` and
  the prototype's Wally on the PATH for the rest, as `just install-local`
  leaves it. The prototype is switched by environment variables
  (`TOWN_BAKE`, `TOWN_FOLD`, `TOWN_MAPS`) and its log lines start with
  `TOWN`; the per-map numbers are the converter's log
  (`build/release/converter.log`): the panorama's way, its tiles and
  colours, the events on screen against the slots, the pictures in front
  and on the wrong side.
* **The pieces**: a throwaway script over the dump's `data/maps/*.json`
  and `db/commonevents.json`, and the CharSets, for the poses' shape.
  "Nobody names it" is `MoveEvent`, `SetEventLocation`,
  `SwapEventLocations`, `FlashSprite`, `ShowBattleAnimation` or a branch
  on its facing, in the map's pages or a common event.
* **Played**: the ports' `tests/objects_per_line.py` (`just
  objects-per-line`) over the whole game, on three cartridges: the 28
  towns baked, the other 27 maps that went over on 2026-11-17 baked, and
  the 55 together; and a
  door's teleport into each town and 240 ticks, counting the frames
  between two `ticks` (Hurón's sweep for the first study). "Today" for
  the lines is the report of 2026-11-17
  (`desert-nightmare-characters-cut-on-crowded-lines`), at the-advocate
  `7dab7d82`; for the frames per tick, the first study's.
* **The colours**: the composite the prototype paints, saved before the
  conversion, against the painting Adeleine decided, pixel by pixel, the
  mean Euclidean distance in 8-bit RGB on every second pixel of every
  second line. Maps with a map picture drawn into the panorama after the
  pieces —98 of them, the night maps among them— are left out: the
  difference there is the picture.
* **The cells**: the decided paintings cut into 8x8 cells, distinct by
  their colour-memory entries (no flips).
* **Not measured**: the vblank time of uploading a column through an
  index; posed over a painting; day 2 onwards and what scenes add after
  the first ticks; bsnes; the night maps' tint, which the teleport
  clears; whether the front mask is right for a character beside a piece
  —it draws the 4 px of the neighbour's sprite that overlap the piece's
  top half under it, where RM2k draws the one to the right over it—.
* **Captures**: EasyRPG Player against the port today and the prototype,
  on maps 2, 3, 4, 5, 14 and 62, entered at the same cell; and every map
  played with the prototype.
