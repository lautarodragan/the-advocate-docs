# What a bush does not model

## What

A terrain's `bush_depth` draws the lower part of a character standing on it
at half its opacity: in RM2k —easyrpg's `Sprite_Character::Draw`—
`GetHeight() / (4 - depth)` rows from the feet, 10 of 32 at depth 1, 16 at
depth 2 and all 32 at depth 3, from the cell the character is on, which
changes as its step begins. The console does it with its colour math —see
`bush_oam` and `bush_blend` in `runtime/terrain_steps.s`, in
the-advocate—, and not all of it:

* **Only the hero.** An event standing in a bush is drawn whole.
* **Only on a map whose characters fit in sprite palettes 0 to 3.** The
  colour math blends every object in palettes 4 to 7 at once, so the
  hero's lower rows go again as 8x8 objects in front of it, over the same
  tiles, in a palette of 4 to 7 that holds a copy of the hero's. A map
  whose characters take more palettes would have them blended too while the
  hero stands in the bush, so there the feet stay opaque. Of the maps of
  the nine games with a bush, measured on 2026-11-01 over the built
  cartridges: The Book of Three's 57 and 84 and Vampires Dawn's 351 and 435
  get it; The Book of Three's 56, *Tylwyth Teg*, which has most of its
  water, takes eight palettes, Vampires Dawn's 201, 236 and 240 and A
  Blurred Line's 5 take seven. Vampires Dawn's forests are on its world
  map, which the cartridge does not carry.
* **Nor while anything else is in 4 to 7**: a picture up, a battle
  animation drawn; and the palette of 4 to 7 it takes is one the timer, the
  rain or the window's objects do not use. With all four taken, opaque.
* **Depth 1 is 8 rows, not 10**: an object repeats the hero's tiles a
  whole tile row at a time, and 32 / 3 is not a tile boundary.
* **What shows through is the map's two layers**, the subscreen, as for a
  half-transparent picture: an event behind the hero's feet does not.

## Why

The colour math is the console's only way to draw half of an opacity, and
it is per layer: for the objects, palettes 4 to 7, all of them. Limiting it
to the hero's feet would need a window and an HDMA channel to say on which
lines, and on the map the six channels are taken while a message is up
—see `registers.s`—; or the leader's palette in 4 to 7, which the
pictures and their colour math use from the top. Neither was done: the
first version draws it where it costs nothing to anyone else.

## How it applies

What it costs, counted from the code: four or eight more 8x8 objects with
the hero in a bush —none at depth 3, which only changes the hero's
palette—, no VRAM, one sprite palette and a 32-byte DMA to it in each
vblank, and the terrain of the hero's cell looked up once per frame. See
[`terrains-encounter-rates-and-bushes`](https://github.com/lautarodragan/the-advocate-work/blob/main/tasks/terrains-encounter-rates-and-bushes.md).
