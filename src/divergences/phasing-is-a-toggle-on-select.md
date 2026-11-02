# Phasing is a toggle on Select

## What

RPG_RT's test play walks the hero through walls and events **while Ctrl
is held**. A port with `[debug] phasing` in its project file does it
**from one press of Select to the next**, and the hero blinks —four ticks
of every sixteen not drawn— while it is on. It is `runtime/phasing.s` in
the-advocate. taro asked for it on 2026-11-02, for Zounds (translated):

> Maybe we could use it to toggle phasing on/off, no?

## Why

Select is taro's choice, and nothing in the runtime reads it: A and B
are RM2k's, X is A's autofire and L and R the hero's speed. Y is free too,
and could be held as Ctrl is; a toggle walks long stretches without
holding anything, and it is what was asked for. The blink is because taro
plays on the console, where there is no other way to see which state it
is in.

## What it does, and what is RPG_RT's

What it lets through was read from easyrpg-player, not from memory:
`Game_Player::UpdateNextMovementAction` calls `SetThrough(debug_flag &&
IsPressed(DEBUG_THROUGH))` around the pad's `Move`, and
`Game_Map::CheckOrMakeWayEx` answers yes to a `through` character right
after checking that the cell exists. So, as in RPG_RT:

* **the tiles, the tile events and the events on the hero's layer let him
  through**; **the map's edge does not**, unless the map loops;
* **only the pad's step goes through**: a forced move route walks with its
  own `ThroughOn` and `ThroughOff`;
* stepping on a cell **still fires its touch pages** below and above the
  hero, and a bump no longer does —no step is refused but at the edge—;
* events walking into the hero are still stopped by him;
* **no random encounters** while it is on: `UpdateEncounterSteps` returns
  first thing. The steps still count for the states' damage.

The other difference: RPG_RT's lasts as long as the key, and this one
outlives a teleport, a battle and the menu. Reset turns it off, and the
save does not carry it. RPG_RT's vehicles take it too; the runtime has no
vehicles.

## How to reproduce it

In Zounds' debug room, press Select and walk up into the trainer or left
into the wall. `tests/test_zounds_phasing.py` in the-advocate-ports does
it, and turns it off again.
