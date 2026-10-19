# What items in battle do not model

## What

The battle's Use Item is RM2k's `Game_BattleAlgorithm::Item` for medicines
and, since 2026-10-19, `Scene_Battle::AssignSkill` with the item for special
items: the item casts its skill —the target by the skill's scope, the damage
or the heal as if the actor cast it— and spends the item instead of the SP.
What goes in the list is `item_in_battle` in `rm2k-snes-battle`, the model
the battle tests predict the exact HP with; the console's side is
`runtime/battle_items.s` in the-advocate. What is not the same yet:

* **The list only shows what can be used.** RM2k's `Window_Item` lists the
  whole inventory and greys out what cannot be used; here the rest does not
  show. A special item does not show either for an actor its `actor_set`
  leaves out, which RM2k greys out per actor.
* **The medicines that only remove conditions do not go in**: the actors
  carry no conditions in battle —see
  [What battle conditions do not model](what-battle-conditions-do-not-model.md)—.
* **A special item casts only what a skill does in this battle**: a
  `normal` skill —its HP, SP, parameters and attribute ranks— or a `switch`
  one with `occasion_battle`. Giving or taking conditions, absorbing and
  reviving are not done, as with any skill; RM2k3's `subskill` is not
  offered.
* **Its message is the item's line alone** —"Kersh Carbon Battery
  used"—: RM2k's next lines, the result on each target, are not written,
  as with any skill here. Nor is the skill's battle animation played, which
  no skill plays.
* **An item with more than one use** —`uses` above 1, Bag of Volatile
  Shells' five— is spent whole on the first one, in battle and on the map:
  RM2k counts its uses in the save (`item_usage`), which this port does not
  carry. One with `uses` 0 is never spent, as in RM2k.
* **The item is spent on choosing it**, not when its turn comes: otherwise
  the last one could be handed out to two actors in the same turn. A failed
  escape gives it back.
* **Equipment that casts a skill** —a weapon or an accessory with
  `use_skill`, which RM2k adds to the list of the actor wearing it— is not
  offered.

## Measured

Against easyrpg-player's source, on 2026-10-19: `Window_Item::CheckEnable`,
`Game_Party::IsItemUsable`, `Scene_Battle::ItemSelected` and `AssignSkill`,
and `Game_BattleAlgorithm::Skill::vStart` and `GetStartMessage`. A Blurred
Line's `test_battle_special_items.py`, in the-advocate-ports, throws a
Carbon Battery at a Junkbot and gives a Cloudy Potion to an ally, against
the model and against easyrpg's arithmetic; the ports'
`test_zounds_special_items.py` has Kazar throw Zounds' Piedra at a Pez
Shark, with the same two checks.

## When it stops being enough

When a game leans on what a special item's skill does beyond its damage:
Vampires Dawn's Itching Powder and The Book of Three's Runes only give
conditions, so in battle they are spent and do nothing. And the bags with
several uses of The Book of Three. The drafts, in `tasks/` in
the-advocate-work:
[`items-that-give-a-condition-in-battle-only-hurt`](https://github.com/lautarodragan/the-advocate-work/blob/main/tasks/items-that-give-a-condition-in-battle-only-hurt.md),
[`an-item-with-several-uses-is-spent-whole`](https://github.com/lautarodragan/the-advocate-work/blob/main/tasks/an-item-with-several-uses-is-spent-whole.md)
and
[`a-skill-in-battle-writes-no-result-line`](https://github.com/lautarodragan/the-advocate-work/blob/main/tasks/a-skill-in-battle-writes-no-result-line.md).
