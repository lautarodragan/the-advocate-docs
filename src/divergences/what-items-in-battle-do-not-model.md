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
* **The item is spent on choosing it**, not when its turn comes: otherwise
  the last one could be handed out to two actors in the same turn. A failed
  escape gives it back. Spending it is RM2k's `ConsumeItemUse` all the
  same: an item with `uses` above 1 —the Bag of Volatile Shells' five—
  counts its uses in the save, as RM2k's `item_usage`, and loses one of the
  count when they reach `uses`; one with `uses` 0 is never spent. So
  choosing it counts a use, and the failed escape gives that use back.
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
Shark, with the same two checks. The uses, on 2026-10-20 against
`Game_Party::ConsumeItemUse` and `AddItem`: the ports'
`test_book_of_three_item_uses.py` throws the Bag of Volatile Shells five
times and gives a throw back with a failed escape.

## When it stops being enough

When a game leans on what a special item's skill does beyond its damage:
Vampires Dawn's Itching Powder and The Book of Three's Runes only give
conditions, so in battle they are spent and do nothing. The drafts, in
`tasks/` in the-advocate-work:
[`items-that-give-a-condition-in-battle-only-hurt`](https://github.com/lautarodragan/the-advocate-work/blob/main/tasks/items-that-give-a-condition-in-battle-only-hurt.md)
and
[`a-skill-in-battle-writes-no-result-line`](https://github.com/lautarodragan/the-advocate-work/blob/main/tasks/a-skill-in-battle-writes-no-result-line.md).
