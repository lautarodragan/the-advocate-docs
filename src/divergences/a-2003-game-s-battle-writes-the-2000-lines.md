# A 2003 game's battle writes the 2000 lines

## What

An RPG Maker 2003 game is fought with the console's battle, which is RPG
Maker 2000's: a hit is told by two lines in the message window,
`<attacker> attacks!` over `<target> 7 HP lost.`, and an item, a defence or
a miss by theirs. RPG Maker 2003's battle never writes those: it tells a
hit by a number that floats over the target, and only names the skill or
the item. So the nine terms of those lines —`attacking`, `defending`,
`actor_critical`, `enemy_damaged`, `enemy_undamaged`, `actor_damaged`,
`actor_undamaged`, `use_item` and `hp_recovery`— were never seen by a 2003
game's author, and what its database holds in them is not the game's
words: in Zounds, the editor's Japanese defaults, which the console wrote
as `Kazar 3'I?_???[?W…` until 2026-10-19.

The converter takes each of those, in a 2003 game, from the project's
`[battle.terms]`, by liblcf's name; and the ones the project does not give,
from plain English stand-ins —` attacks!`, ` HP lost.`…—, as easyrpg
falls back to English for its own 2003 battle lines. A 2000 game's terms
are its database's, unless `[battle.terms]` gives them too. See
`UNSHOWN_IN_2003` and `battle_term` in `rm2k-snes/mapdata/src/emit.rs` in
the-advocate.

## What RM2k3 does

`Game_BattleAlgorithm::*::GetStartMessage`, in easyrpg, writes the 2000
lines only `if (Feature::HasRpg2kBattleSystem())`; with the 2003 battle,
`Scene_Battle_Rpg2k3` shows the skill's or the item's name at the top and
the damage as a number over the battler.

## What is not there

* **The floating numbers** and the rest of the 2003 battle: see
  [Zounds, an RPG Maker 2003 game](../research/2026-10-15-zounds-an-rpg-maker-2003-game.md),
  section 3.
* **The `¡`** in a Spanish game's lines: each term goes after a name or a
  number, so it can only end the line, not open it.
