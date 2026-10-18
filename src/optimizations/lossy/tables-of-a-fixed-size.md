# Lossy: tables of a fixed size

## Graphics a move route switches to

A map holds a fixed number of the graphics move route command 34 switches
to; the rest are ignored.

- **Loses**: those changes of graphic.
- **Where**: `ROUTE_GFX_MAX` in `rm2k-snes/events/src/registry.rs` and, the
  same number, in `rm2k-snes/mapdata/src/record.rs`.
- **More**: [Move route command 34 on the hero](../../decisions/move-route-command-34-on-the-hero.md).

## Events that walk by a movement type, past the route slots

An event whose page walks by a movement type —random, toward the hero,
back and forth— walks on one of the map's route slots, the same ones a
`MoveEvent` uses, and takes one only while a few stay free for those. On
a map with more walkers than that, the rest stand at their cell.

- **Loses**: those events' walking.
- **Saves**: bank 0's RAM —a route slot per walker on the map— and a
  `MoveEvent` that finds no slot.
- **Where**: `walk_may_take` and `WALK_RESERVE` in
  `runtime/walk_by_type.s`.
- **More**: [What walking by a movement type does not model](../../divergences/what-walking-by-a-movement-type-does-not-model.md#when-it-stops-being-enough).

## Variables

The console keeps a fixed number of variables, each with a one-byte slot.
Event pages that need one that didn't get a slot are left out, and the
build lists them.

- **Loses**: those pages.
- **Where**: `variable_slot` in `rm2k-snes/events/src/compile.rs`.
- **More**: [Variables saturate at sixteen bits](../../divergences/variables-saturate-at-16-bits.md).

## A troop of more than eight enemies

A battle holds eight enemies: each takes a fixed place in the battle's far
RAM, and the hidden and scene masks are a byte per troop. A troop with
more is fitted to eight, leaving out first the members no page shows, then
those every page that shows them also kills; never one a page condition
or a branch asks about.

- **Loses**: those members, in the scenes the troop's pages play.
- **Where**: `fit.rs` of `rm2k-snes/battle/`.
- **More**: [A battle holds eight enemies](../../divergences/a-battle-holds-eight-enemies.md).
