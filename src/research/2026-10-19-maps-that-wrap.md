# Maps that wrap around

*2026-10-19. Measured on the-advocate `2c350d8f` and the-advocate-ports
`1182513`, every game of the ports; what RM2k does, read in easyrpg-player
`9e67927` and liblcf `6854310`. It is a snapshot: it is not updated.*

taro, playing Zounds: "`Subsuelo / Passage 1` has `Wrap: Both` on. We do
not have that implemented, do we? How complicated would it be?" No, it is
not implemented, and nothing says so: the converter reads the field and
drops it, and the console clamps the map at its edges. This page says what
RM2k does with it, which maps of the ports use it and for what, and what
it would cost. The task:
[`zounds-a-map-that-wraps`](https://github.com/lautarodragan/the-advocate-work/blob/main/tasks/zounds-a-map-that-wraps.md);
the implementation, with the scope proposed here:
[`maps-that-wrap-around`](https://github.com/lautarodragan/the-advocate-work/blob/main/tasks/maps-that-wrap-around.md).

## What RM2k does

The map's `scroll_type` (chunk `0x0B`) is `none`, `vertical` (1),
`horizontal` (2) or `both` (3) —note the order: 1 is vertical—. It makes
the map a torus on that axis, and everything that walks on it agrees.
In easyrpg-player it is `Game_Map::LoopHorizontal` and `LoopVertical`, and
`RoundX`/`RoundY`, which take a coordinate modulo the map's size:

* **The step.** `Game_Character::Move` asks for the way with the
  coordinate *before* rounding, so the faces are the usual ones —the
  cell being left lets one out on that side, the one being entered lets
  one in on the other—, and only then rounds the target
  (`CheckOrMakeWayEx`). Crossing the seam is an ordinary step between
  column W-1 and column 0; a seam a wall closes stays closed. The hero
  and the events alike, routes, walking by type and the walk-through
  flag included.
* **Jumps** land rounded, and the start of the jump is moved by the same
  amount so the arc keeps its direction (`Game_Character::Jump`).
* **Distances** to the hero —move toward him, away from him, face him—
  take the short way round: a difference past half the map is folded
  (`GetDistanceXfromCharacter`). The cell in front of someone, for the
  action key and for touch, is rounded too (`XwithDirection`).
* **The camera** does not stop at the edges: `SetPositionX` and
  `AddScreenX` keep it modulo the map's width instead of clamping it. The
  hero stays centred and the other side of the map shows past the edge.
  On a map of 20x15 —exactly RM2k's screen— the screen scrolls with the
  hero over a map that repeats.
* **The drawing**: the tile layers repeat (`TilemapLayer`); each character
  is placed modulo the map's size (`GetScreenX`/`GetScreenY`), and gets up
  to three clones, so that one standing on the seam shows on both sides of
  it (`Spriteset_Map::CreateSprite`).
* **The panorama**, on a looping axis, does not follow the map: the
  panorama-follows-the-camera rule of a non-looping map is skipped
  (`Parallax::ScrollRight`/`ScrollDown` return), and only its own
  automatic scroll moves it. It is drawn tiled across the whole screen.
* **Terrain** under a coordinate outside the map is rounded on a looping
  map (`GetTerrainTag`).
* What is **not** rounded: a `SetEventLocation` or a teleport to a cell
  outside the map. RPG_RT leaves it out of bounds until the next step;
  easyrpg rounds it on the first one.

## What the console does today

**Nothing, and silently.** Ditto reads `scroll_type` into the schema, and
nothing in `rm2k-snes/` or `runtime/` looks at it: the converter's log and
`degradations.json` say nothing about it, and the map is converted as any
other. On the console every edge is a wall —`cell_neighbor_exists` in
`runtime/cells.s` and the hero's `try_*` in `runtime/hero.s` refuse the
step that would leave the map—, and `follow_camera` clamps the camera to
`cam_max_x`/`cam_max_y`.

Played on Zounds' release ROM, with the ports' harness: the hero put next
to each of map 70's four seams and walking into them for 60 to 80 ticks
stops at x 99, x 0, y 0 and y 49, with the camera clamped
(`70-*-walked-*.png` in the captures). Nothing hangs; he simply cannot go
on.

## The maps that wrap

Twelve maps in the ports, in three games; none in A Blurred Line, Book of
Three, Desert Nightmare, Dooms 2, El Dorado or The Frozen World. "Crosses"
is whether the hero can walk across a seam: a cell of the last column
that lets one out to the right whose neighbour of the first column lets
one in from the left, reached walking from where the game puts the hero
—with the doors inside the map as passages, and the events left out of
the passability—.

| Game | Map | Size | Wraps | What it is | Crosses |
| --- | --- | --- | --- | --- | --- |
| Zounds | 70, Passage 1 | 100x50 | both | a maze of corridors that go on across both seams | yes: 5 rows, 3 columns |
| Zounds | 30, Sueño 2 (Hover Tram) | 200x200 | both | the dream: two minutes chasing the boss | yes: 4 rows, 2 columns |
| Zounds | 103, Cuarto 2 | 35x45 | both | the room behind map 70's seam | no |
| Zounds | 137, Secret Entrance | 30x25 | both | corridors that go on across both seams | yes, open on both |
| Zounds | 26, Explicación de Desierto | 20x15 | both | a cutscene over a desert that repeats | open everywhere, but the hero takes one step |
| Zounds | 27, Montaña | 30x22 | both | a cutscene | no |
| Zounds | 150, Scape Way | 20x15 | vertical | a corridor the hero walks up forever while the party talks | yes, 3 columns |
| Vampires Dawn | 1, Weltkarte | 300x300 | both | the world map | no; and the map is left out of the cartridge |
| Vampires Dawn | 91, Asgars Wald | 60x60 | both | a forest "too big", a maze with dogs | yes, open nearly everywhere |
| Vampires Dawn | 249, Teil43 | 20x100 | vertical | a cutscene, the hero flying up | the route would, but the scene ends first |
| Vampires Dawn | 254, Freier Fall Stufe 1 | 20x25 | vertical | a free fall: the hero walks down forever, the stones up | yes: everything moves forever |
| Sunset Over Imdahl | 63, TownWall(Night) | 30x15 | horizontal | the town wall at night: bats fly left forever, walk-through on | not the hero; the bats do |

They are of three kinds:

1. **A place the hero walks across.** Zounds 70, 30 and 137, Vampires
   Dawn 91. On **Zounds 70 it costs a room**: the door at (86,13) to map
   103, *Cuarto 2* —its chests, the one with the Silver Key that opens a
   door on map 59 among them, and its bosses— is reached only through a
   seam. Walking from where map 67 leaves the hero, 1,263 cells are reached without wrapping and 1,333
   with it, and the door is among the 70. On the console nobody reaches it
   walking. Whether the key is on the way of the story was not looked at.
   On the dream, map 30, wrapping opens 638 more cells, the teleport
   corridors at (21,84) and (127,183), but the dream ends by its timer
   either way. On 137 and 91 the maze is whole without the seams; what is
   lost is the feeling of a place without end, and near the edges the
   camera.
2. **Something that moves forever.** Vampires Dawn 254 —the hero falls
   down a 25-row map by a repeating route, the stones rise by theirs—,
   Sunset 63 —the bats walk left with walk-through on, by repeating
   routes— and Zounds 150 —the hero walks up by a repeating route while
   the party talks—. Without wrapping, the falling hero of 254 stops at
   the bottom after some twenty steps, and the stones and the bats pile
   up at their edges: the
   minigame and the scene stop looking like themselves. It is the kind
   that most needs the events, not only the hero, to wrap.

   Two of them need a second thing: the hero's walk on Zounds 150 and
   on Vampires Dawn 254 is a `MoveEvent` with "repeat", which the
   console runs once —the loss `route-repeat-dropped` in
   `degradations.json`, task
   [`a-forced-routes-repeat-flag-is-dropped`](https://github.com/lautarodragan/the-advocate-work/blob/main/tasks/a-forced-routes-repeat-flag-is-dropped.md)—.
   On 254 the parallel page that gives it issues it again every few
   frames, which hides it; on 150 an automatic page gives it once and
   erases itself, so today the hero takes **one** step —read in the
   data, not played—, and wrapping alone would not change that.
3. **Only the camera.** Zounds 26 and 27 and Vampires Dawn 249 are
   cutscenes whose routes do not reach a seam, and Zounds 103 has no
   open seam; Vampires Dawn 1 is not in the cartridge. What changes
   there is that RM2k's camera does not stop at the edge: Vampires Dawn 249 starts with the hero on row 98 of 100,
   and RM2k shows the map's top rows below him.

The sizes matter for one detail: every wrapping map is at least 20
cells wide, 320 pixels, so a character is never seen on both sides of a
vertical seam at once on the console's 256-pixel screen; RM2k's clones are
not needed across. The two that wrap vertically with 15 rows, 240 pixels, are 16 pixels
taller than the screen, so a 32-pixel character on the horizontal seam
could be: on both, Zounds 26 and 150, no event is drawn, and the hero is
the camera's centre.

The only panorama on a wrapping map is Sunset 63's, and it is 320x240,
the screen of RM2k: there it does not move with or without wrapping.

## What it would cost on the console

**The tilemap is already a torus.** `runtime/map_window.s` keeps a window
of 64x32 cells in which cell (x, y) of the map always lives in slot (x mod
64, y mod 32), and rewrites the column or row that comes on screen; the
hardware scroll is ten bits, and 1,024 is a multiple of the window's 512
and 256, so the camera is written to it as is. What does not wrap is what
feeds it:

* **The camera has to be continuous.** If the camera's position jumps by
  the map's width when the hero crosses the seam, the slot of each cell
  jumps with it —the map's width is not a multiple of 64—. So the view
  that the window and the scroll registers follow is a position that only
  moves by the camera's change, folded to the short way round, and the
  camera in map coordinates is that view modulo the map's size. Without
  clamping on a wrapping axis: `cam_max_x`/`cam_max_y` already come per
  map in the record, and a value that says "wraps" costs no byte.
* **The strips read the map modulo its size.** `build_column` and
  `build_row` walk the grid with a pointer and stop at the map's edge,
  filling the rest of the strip with nothing; on a wrapping axis they go
  on from the first row or column instead —a second stretch with the
  pointer reset—. Which map column a window column is cannot come from a
  division —a map's width in 8x8 cells passes 255, Zounds 30 has 400, and
  the divider takes eight bits— but it does not need one: the window advances
  one column or row at a time (`one_column`, `one_row`), so the map column
  advances with it and wraps at `map_cw`. A full refill —entering, a
  teleport— starts the view at the camera.
* **Stepping and passability.** `cell_neighbor_exists`, which every event
  and route goes through, says yes across a wrapping edge; the cell
  pointer, which today moves by one cell or one row, is computed again
  when the step crosses the seam; and the coordinates of the cell being
  entered —`add_a_to_dest_x` and `add_a_to_pass_x` and their Y twins—
  wrap. The hero's four `try_*` in `runtime/hero.s` are the same change
  four times, and `can_fall` the same for a jump. Collision with someone
  standing on the target needs nothing more: it compares the cell, already
  wrapped.
* **Pixel positions.** The hero's and the events' positions in pixels are
  the cell times 16 minus what is left of the step, so for the step that
  crosses the seam they leave the map by up to 16 pixels. Every place that
  subtracts the camera from a position —the characters in
  `runtime/oam.s`, the hero there too, `in_sight` in
  `runtime/walk_by_type.s`, a battle animation over someone— folds the
  difference into the map's width. Distances to the hero
  (`turn_toward_hero.s`, toward and away in `walk_by_type.s`) fold the
  same way.
* **The order of drawing** on a vertical seam: the characters are kept
  sorted by Y for the OAM, and one at row 0 under the camera that shows
  row H-1 above it sorts wrong for the few frames both are near. No map
  of the ports has drawn characters crossing each other there.
* **Not needed for the ports**: clones of a character on both sides of a
  seam (see above), the panorama on a looping axis (Sunset's does not move
  either way), pictures tied to the map.

**Where it goes.** Bank 0 is the tight one: in the release cartridges of
that day it had 218 bytes left in Zounds, 128 in Vampires Dawn, 108 in
Sunset and 111 in A Blurred Line. Counting the routines above, the change
is some 500 to 700 bytes of code, an estimate and not a measure. It does
not fit in bank 0, and it does not have to: the hooks in bank 0 are a
`jsl` each to code in a far bank, and the whole of it is assembled only
into a cartridge that carries a wrapping map, as `paint_table` assembles
the painted maps' code only where there is one —A Blurred Line, which has
none, would not change by a byte—. RAM: about a dozen bytes —the view in
X and Y, the map column and row the window is at, the map's size in
pixels—, in `FARBSS`; `BSS` had 81 bytes left in each of the four.

**The cheapest scope that covers the ports** is both axes for the hero
and the events —steps, routes, walking by type, walk-through, jumps,
distances—, the camera without clamping and the window streaming across
the seam; without clones, without the panorama rule and without pictures.
That covers the room on Zounds 70, the free fall of Vampires Dawn 254, the
bats of Sunset 63 and the endless corridor of Zounds 150, and the rest
comes with it. Before it, and on its own, the converter can say it: a
`map-does-not-wrap` entry in `degradations.json` for each wrapping map,
which is a few lines and tells the next one who plays it what to expect.
