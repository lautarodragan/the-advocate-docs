# Still events drawn into the background

## What

With `[graphics] posed_events = true`, a still event that only changes
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
  front of it, who covers it in RM2k too.
* **A message without its background does not take it off its lines**,
  as it takes the layer above (see [The window without a background does
  not cover the characters](the-window-without-a-background-does-not-cover-the-characters.md)):
  it is the background, as the floor under it.
* **Many at once can take a frame more.** A cell is two runs of tilemap
  words, a DMA each, and a vblank takes sixteen runs of each row of cells:
  cells side by side join one run, but past that the rest go in the next
  frame, each cell whole.

**Which events**: every page that draws draws a CharSet, or a ChipSet
tile that does not go above the hero; no page walks, animates standing
still or puts it above the hero; nothing moves it —a route that steps,
`SetEventLocation`, `SwapEventLocations`—, flashes it —`FlashSprite`— or
gives it another graphic; it does not turn toward the hero —the console
draws that one from the turn slot—; no substitution swaps its cell; and
it is not translucent. Routes that only turn it are allowed: the turn is
a pose. The map shows no panorama, draws no pictures into itself and
does not loop. And it stays an object if, with its poses, the background
merges colours of the ground or spends more tiles than the layouts leave:
all of them, or CharSet by CharSet, the most numerous first. The
converter's log says, map by map, which went in and why each of the rest
did not.

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
7, but one of 17 looked at, 8.

## When it stops being enough

* A chest that turns toward the hero when opened, or whose lid stands
  above its cell, stays an object: none of Zounds' chests goes in.
* A whole map's switches turning together: past sixteen runs a vblank
  the last ones wait a frame.
* A map with no room for the poses' tiles or colours keeps them as
  objects.

## How to reproduce it

`wally map` on the map's request with `"posed_stills": true`; the
runtime is `posed.s` in the-advocate. Zounds turns it on.
