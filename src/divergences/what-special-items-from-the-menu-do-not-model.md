# What special items from the menu do not model

## What

A special item —RM2k's `Type_special`, one that casts a skill— is used from
the pause menu's items screen as RM2k's `Scene_Item` uses it, since
2026-11-01: white if the map can cast its skill —`Algo::IsSkillUsable`,
`usable_on_the_map` in rm2k-rules—; a switch skill turns its switch on,
spends the item and goes back to the map, as a switch item does; any other
opens the actor choice as a medicine's, with the item's name and how many
are left, the bar over everyone for a skill on the party and free for one
on oneself. A casts the skill with the attack and spirit of the highest
level in the party who can act and whom the item's `actor_set` lets use it
—`Game_Party::GetHighestLeveledActorWhoCanUse`—, or of the one chosen for a
skill on oneself, and spends the item only if it was of use. Read from
easyrpg-player's `Scene_Item`, `Scene_ActorTarget`, `Game_Party::UseItem`
and `Game_Party::IsItemUsable`. What it does differently:

* **The grey does not move.** For an item whose skill is a normal one,
  RPG_RT also greys it while nobody in the party can act and use it; here
  the list is white or grey by the database alone, and A on such an item
  does nothing, which is where RPG_RT would not have let it be chosen.
* **The effect is the skills screen's**, with what
  [What healing skills from the menu do not model](what-healing-skills-from-the-menu-do-not-model.md)
  says of it: the battle's randomness, and no sound —neither the skill's
  animation's on a use nor the buzzer, nor a switch skill's sound effect,
  the draft
  [`switch-skills-from-the-menu-play-no-sound`](https://github.com/lautarodragan/the-advocate-work/blob/main/tasks/switch-skills-from-the-menu-play-no-sound.md)—.
* **Teleport and escape skills stay grey**, as on the skills screen: the
  map does not know where they would take the party. No game of the nine
  has one on an item.

The switch skill's path has never run: the only special item with one in
the nine games, Zounds' Materializador (item 7), is not given by any of its
maps, so no cartridge carries it.

## How it applies

The grey and whether it opens the actor choice are `item_usable` and
`item_needs_actor` in `rm2k-snes/screens/src/texts.rs`, the converter's
`item_map_skill` table comes from `item_skill_on_the_map` in
`rm2k-snes/battle/src/rules.rs`, and the use is `item_cast_use` in
`runtime/item_skills.s`, which casts with the skills screen's `sk_cast`, in
the-advocate. The tests are `tests/rpgmaker/test_menu_special_items.py` of
A Blurred Line, over its Cloudy Potion. The task is
[`items-that-cast-a-skill-outside-battle`](https://github.com/lautarodragan/the-advocate-work/blob/main/tasks/items-that-cast-a-skill-outside-battle.md).
