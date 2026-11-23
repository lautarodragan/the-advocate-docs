# What `ChangeMapTileset` does not model

## What

`ChangeMapTileset` (11710) has RPG_RT draw the map, from that command until
the next map is loaded, with another ChipSet: the same cells, with that
ChipSet's graphics, its passability, its layer above the hero and its
animated tiles —`Game_Map::SetChipset` in EasyRPG—. Since 2026-11-23 the
console does it for the ChipSets **a map's own pages** ask for: the
converter converts the map once more with each, over the same cells, and
the command swaps that conversion in. What it does not model:

* **A swap in a common event, or after a `Teleport` in the same page**, is
  dropped, as every swap was until that day: the map keeps its ChipSet, the
  page goes on, and the build lists a `tileset-not-changed` loss in its
  `degradations.json`. A common event knows no map, so the converter cannot
  know which maps to convert with that ChipSet. No game of the ports has one;
  the test game's case `tileset` does, on purpose.
* **A map that cannot take a ChipSet that draws otherwise** keeps the loss
  too, and the converter's log says why: a map with a panorama, with a
  `TileSubstitution` or with still events drawn into its background, or one
  whose events come out drawn otherwise with the other ChipSet. None of the
  ports' maps that swap is one of those.
* **A ChipSet with the map's own picture and animation only changes what
  lets through**: its passability and its terrain. What it puts above the
  hero stays where the map's own has it. Vampires Dawn's bat is that: every
  cell opens, and what the castle draws above the hero stays above the bat
  and the rest below it.
* **An event drawn with a ChipSet tile keeps the map's ChipSet's drawing**,
  unless it is baked into the map's layers, which are drawn again.
* **The screen goes black for the tick of a swap that draws otherwise**,
  while the ChipSet's tiles and colours are uploaded: RPG_RT swaps it from
  one frame to the next. In The Frozen World every swap falls inside a
  flash or a scene.
* **A saved game comes back with the map's own ChipSet**: RPG_RT saves the
  one in use. The swaps of the ports are made by parallel pages that ask
  again —Vampires Dawn's bat, every frame its switch is on— or inside
  scenes.

## Why

Until 2026-10-16 the command had no arm in the event compiler and its whole
page was left out; in The Frozen World the intro's step on map 9 is one of
those pages, and the game stopped there. From then until 2026-11-23 the
command was dropped and the map kept its ChipSet: Frozen World's bazaar and
temple kept their water moving where the original stops time.

The console cannot draw a map with a ChipSet it was not converted with, so
each one a map swaps in is a **variant** of the map: its own tiles, colours
and animation, and its own block table over the map's grid. The blocks are
numbered by every ChipSet at once, so the table has the same size whatever
the ChipSet and lives where the map's does, behind the grid, with the
event bytecode behind it untouched. What it costs and the alternatives were
measured on 2026-10-16 in the task
[`changemaptileset-is-not-compiled`](https://github.com/lautarodragan/the-advocate-work/blob/main/tasks/changemaptileset-is-not-compiled.md);
what was done, in
[`changemaptileset-is-not-done`](https://github.com/lautarodragan/the-advocate-work/blob/main/tasks/changemaptileset-is-not-done.md).

## How it applies

`variants.rs` in `rm2k-snes-convert` decides the variants and numbers the
blocks; `variant_data` and `tileset_table` in `record.rs` of
`rm2k-snes-mapdata` give each a record of `map_table` behind the maps';
the command's arm in `compile_command`, in `compile.rs` of
`rm2k-snes-events`, numbers them; and `runtime/tileset.s` swaps one in, all
in the-advocate. The test game's map 16, the frost, swaps its ChipSet three
ways —`test_tileset_swap.py`—, and the case `tileset` calls a common event
that asks for one —`test_tileset.py`—. In the ports,
`test_frozen_world.py` watches the bazaar's background stand still after
the flash.
