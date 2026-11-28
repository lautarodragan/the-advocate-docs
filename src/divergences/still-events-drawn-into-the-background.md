# Still events drawn into the background

## What

With `[graphics] baked_events = true` —or a list of map ids, only on
those: each map that takes it spends background tiles, and a cartridge
with no room left can pay for some maps and not all—, a still event that only changes
pose —a floor switch, a lever, rubble that a page takes away— is not an
object: its cell gets one block of the background per pose it can show,
and the console writes the cell's place in the grid and its four tilemap
words again when its row's page or a route's turn changes the pose. The
event stays whole —it triggers, blocks, has pages and routes—; only its
drawing moves to BG1. What changes with respect to RM2k:

* **It is always under every character**, also one on its own layer that
  stands above it on screen. RM2k sorts the events by y. An event on the
  hero's layer goes into the background only if every pose it shows stays
  inside its 16x16 cell, so the only character that overlaps it is one in
  front of it, who covers it in RM2k too —or if what it draws past its
  cell lands where nobody RM2k would draw under it can ever be: see
  [Drawn past its cell](#drawn-past-its-cell)—. One beside it on its row,
  to its left, draws its 4 px side over it, where RM2k draws the event
  over that side.
* **A message without its background does not take it off its lines**,
  as it takes the layer above (see [The window without a background does
  not cover the characters](the-window-without-a-background-does-not-cover-the-characters.md)):
  it is the background, as the floor under it.
* **It can stand a pixel or two off from where RM2k draws it.** A pose
  is a 24x32 drawing standing on its cell, and one whose drawing spills a
  little past the 16x16 of the cell —A Blurred Line's chests stand 1
  pixel right of it and 2 above— goes in moved back into the cell,
  whole, nothing cropped: `[graphics] baked_events_nudge` is how many
  pixels it may move, in x and in y: 2 when the project does not say,
  0 to turn it off. Every pose of
  one event moves by the same amount, or a chest would jump as it
  opens: if no single shift fits them all, it stays an object. The same
  graphic on an event that stays an object —on another map, or on the
  same one— is drawn where RM2k draws it, so the two can stand a couple
  of pixels apart. The converter's log says which events moved and by
  how much.
* **Many at once can take a frame more.** A cell is two runs of tilemap
  words, a DMA each, and a vblank takes sixteen runs of each row of cells:
  cells side by side join one run, but past that the rest go in the next
  frame, each cell whole.

**Which events**: every page that draws draws a CharSet, or a ChipSet
tile that does not go above the hero; no page walks, animates standing
still or puts it above the hero; nothing moves it —a route that steps,
`SetEventLocation`, `SwapEventLocations`—, flashes it —`FlashSprite`, or a
battle animation on it: the flash tints the object's palette, and drawn
into the background it has none— or gives it another graphic; it does not turn toward the hero —the console
draws that one from the turn slot—; no substitution swaps its cell; and
it is not translucent. Routes that only turn it are allowed: the turn is
a pose. The map draws no pictures into itself and does not loop. If it
shows a panorama —one it covers everywhere, which goes hidden, is no
panorama—, only with `[graphics] baked_events_over_panoramas` too —`true`
or a list of map ids, off when the key is missing: it spends background
tiles on maps `baked_events` alone leaves as they were—, and then the
panorama goes in mode 3, flattened, mode 1 or baked into the background,
the ways that convert the map with the poses' cells, and every cell the
event is drawn on hides it whole, in every drawing of the map's
animation. Baked, also only if no substitution needs rows of its own
below the map: the panorama is baked under those at their place, not at
the cell they stand for. In mode 1 the map is at 4 bits, and the poses'
colours have to fit its cells' palettes: the colour rule below holds them
to it. Until 2026-11-28 a seen panorama took them under `baked_events`
alone, in mode 3 and flattened. And it stays an object if, with its poses, the background
merges a single colour more —the event would come out a shade off— or
spends more tiles than the layouts leave: all of them, or CharSet by
CharSet, the most numerous first. The
converter's log says, map by map, which went in and why each of the rest
did not.

## Drawn past its cell

A person is 24x32 on a 16x16 cell: RM2k draws its head on the cell above
and its sides 4 px into the cells beside it. Since 2026-11-18 such an
event goes into the background too when:

* **what it draws past its cell is the same in every pose and never goes
  away**: every page draws it, the first asks for nothing —so some page
  always holds—, none erases it or calls a common event that may, and it
  stays on one layer. The console writes only the event's own cell when
  its pose changes; what falls on the cells around it is drawn into the
  map once, back to front in RM2k's order —layer, row, column, id—, and
  each pose's cell carries under it what the events behind it draw onto
  it and over it what the ones in front draw;
* **nobody RM2k would draw under it can be where that is drawn**. Where
  every character's feet can ever be is `standing` in rm2k-rules' `reach`
  —the same walk that says which tiles above the hero nobody can cover:
  the hero, every event, every route that can run, phasing included—. RM2k
  draws a character on the row above under the event, one on its own row
  to its left too, one on its own cell with a lower id —the hero, always—
  too, and one on the layer below always. If any of those can stand where
  its sprite covers a pixel of the spill, the event stays an object.
  **The one to its left is let through**: it already draws its 4 px side
  over every event drawn into its cell, as said above, and the spill adds
  the same 4 px, over the head too;
* **a still event behind it that stays an object** would be drawn over the
  spill: that one has to go into the background too, or this one goes out;
* the spill does not fall on a cell a `TileSubstitution` swaps, nor past
  the map's edge.

For a set with spills, the background may take more entries of colour
memory, and the layer above fewer palettes, if neither merges anything
more: A Blurred Line's Hover-Tram had a background of 35 entries, and its
seated people bring 48 colours.

What `reach` had to learn for it, each of which covers fewer cells than
before and never more —so a game with `unreachable_above_below` lowers
more too—: a page in the foreground that takes the hero off the map
before it can end only *holds* him where its routes leave him; a common
event that walks him while he is held leaves him held; only an event
drawn with a tile opens its cell to the others; a route that is not
skippable waits at a step it cannot take; an autostart page that asks
for nothing, under pages that ask only for switches nothing on the map
turns off, starts where the hero arrives; the hero and the events walk on
their own in four directions; and a scene walks the hero through walls
only from where it left him phasing. The converter, for its part, takes a
recall whose map variable is only set to numbers to those maps only.

Measured on 2026-11-18 with the-advocate's `cardon-hover-tram`, on A
Blurred Line's `release`: 478 events in the background before, 612
after —110 of them drawn past their cells, on 20 maps, and the rest
inside their cells on maps whose panorama shows—, for one more half bank
of 32 KB. On the Hover-Tram only one of the seven seated people goes in:
`reach` lets a scene seat the hero on any seat, since a recall can put him
anywhere on the aisle, and the colours of the others would take the
panorama's. (Since then a patch of the map bakes its whole still crowd:
[The Hover-Tram's crowd is part of the map](../ports/abl/decisions/the-hover-tram-s-crowd-is-part-of-the-map.md).) Zounds: 427 before, 457 after, 22 of them past their cells
on 9 maps, about 12 KB of its gaps and no whole bank. The Frozen World turns
it on only on map 128, its market, since 2026-11-22: its sixteen stalls
and crates put eleven characters on one line as objects, and the hero
walking past them lost her head. The other games do not turn
`baked_events` on.

## Why the background and not the layer above

Mode 3 draws, front to back, objects 3, BG1 1, objects 2, BG2 1, objects
1, BG1 0, objects 0, BG2 0. The map is BG1 0 and the characters objects
1: BG2 has no place between them —at priority 1 it covers the hero, which
is what the layer above is for, and at 0 the floor covers it—. Keeping
BG2 needs the characters raised to objects 2 where they overlap such an
event, and that is right only where no tile of the layer above is in
their drawing at the same moment. On Zounds' map 96 that held for 15 of
its 56 switches: they stand in a corridor between ceilings that stay
above, and the room's bottom edge is drawn over a row of them. The
background has no such limit. The measures are in the task
[`still-events-that-change-pose-painted-into-the-background`](https://github.com/lautarodragan/the-advocate-work/blob/main/tasks/still-events-that-change-pose-painted-into-the-background.md).

## Measured

On Zounds' map 96, *Arriba del Cuartito*, on 2026-11-02, in snes9x, at
seven spots by the switches: 240 frames per 120 ticks before, 120 after;
up to 128 slivers of eight pixels on a line before, 16 after; the main
loop from 284–337 lines to 186–196, of the frame's 262. 68 of its events
go into the background.

The frames from the change to the pose on screen, as objects and in the
background, measured on the pixels: rubble that a switch takes away on
map 114, 2 and 2; the 55 switches that switch 139 turns on map 96, 7 and
7, but one of 17 looked at, 8. Map 96's chest, which stays an object,
opens in 6 frames where it took 8: the map no longer lags.

### The nudge

Measured on 2026-11-16 with the-advocate at
8e670c9e, converting each game with `baked_events = true` and the
nudge at 0 and at 2 —A Blurred Line also at 1, 3, 4 and 8, in a
throwaway build, since its `release` has no room for them yet—:

| Game | In the background, nudge 0 | More with 2 | Shifted by | Data |
|---|---|---|---|---|
| A Blurred Line | 419 | 41 | (−1, +2) ×33, (0, +1) ×7, (0, +2) | +4,726 B |
| The Book of Three | 51 | 0 | | 0 |
| Desert Nightmare | 27 | 5 | (0, +1) ×5 | +990 B |
| Dooms2 | 14 | 0 | | 0 |
| El Dorado | 35 | 2 | (−1, 0), (0, +1) | +952 B |
| The Frozen World | 118 | 0 | | 0 |
| Sunset over Imdahl | 0 | 0 | | 0 |
| Vampires Dawn | 2,809 | 17 | (0, +1) ×16, (0, +2) | +497 B |
| Zounds | 426 | 1 | (0, +2), a chest on map 17 | +375 B |

No event goes out with the nudge: the ones that fit their cell as they
are go in first, and the shifted ones only on top of them. The bytes are
the converter's data, map tiles and the posed record against the sprite
tiles they no longer need; on A Blurred Line about 2.9 KB of them are the
sprite pose bank, which changes with which poses stay sprites.

Where the same graphic is shifted on a map and stays an object on the
same map, the two stand a pixel or two apart: Vampires Dawn's map 148,
El Dorado's maps 2 and 5, Desert Nightmare's 105 and 182. A Blurred Line
and Zounds have none.

The default is 2 because it is where a drawing that is only misaligned
stops: on A Blurred Line, 1 takes 7 events, 2 takes 41 —its 33 chests,
`BLObjs` character 2, which stand a pixel right and two up of their
cell, seven more of `BLObjs` character 3 a pixel or two up, and a
radio—, 3 none more, and 4 a missile drawn four pixels above its cell
on purpose, floating. A Blurred Line's chests that stay objects —moved
by a command, or on a map whose panorama stays— are on other maps than
the shifted ones. Captures
of the shifted ones against where RM2k draws them are in the task
[`posed-events-tolerate-a-few-pixels-of-overflow`](https://github.com/lautarodragan/the-advocate-work/blob/main/tasks/posed-events-tolerate-a-few-pixels-of-overflow.md).

## When it stops being enough

* A chest that turns toward the hero when opened stays an object: none
  of Zounds' chests goes in.
* A person whose head falls where someone walks behind it —a crowd
  standing on an aisle, whose heads are over the aisle— stays an object.
  So does one wherever a scene may seat the hero on its row or walk him
  behind it: `reach` does not know a scene runs once, nor which of the
  places a game recalls the hero to it really uses. See the task
  [`abl-the-hover-tram-s-crowd-is-cut`](https://github.com/lautarodragan/the-advocate-work/blob/main/tasks/abl-the-hover-tram-s-crowd-is-cut.md).
* A whole map's switches turning together: past sixteen runs a vblank
  the last ones wait a frame.
* A map with no room for the poses' tiles or colours keeps them as
  objects.
* A map drawn as a painting, whose panorama is fixed to it, does not take
  them at all: its still events that never change look are painted into
  the panorama instead, the ones drawn past their cells and above the hero
  too, if the project asks for it —see [the still pieces, painted
  in](../internals/vram.md#the-still-pieces-painted-in)—.

## How to reproduce it

`[graphics] baked_events`, `baked_events_over_panoramas` and
`baked_events_nudge` in the project's `advocate.toml`, which reach Wally's
map stage —`stage::map` in `wally`— as `"posed_stills": true`,
`"posed_over_panoramas": true` and `"baked_events_nudge"` in the map's
request; the runtime is `posed.s` in the-advocate. Zounds
turns both on, and A Blurred Line since 2026-11-16, once its event page
tables went to WRAM compressed and the runs left far RAM for `GRID_TOP`
(see [The WRAM map](../internals/wram.md)). Until 2026-11-16 the key was
called `posed_events`.
