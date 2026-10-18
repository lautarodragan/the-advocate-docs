# What attributes do not model

## What

Since 2026-10-15 the battle applies RM2k's attributes —the database's
`Attribute`s, Sword, Flame and the rest— as easyrpg-player's
`ApplyAttributeMultiplier` does for RPG Maker 2000. Of the attributes a hit
carries, the target's rank in each, `A` to `E`, gives that attribute's
rate —`a_rate` to `e_rate`—; within a kind, physical or magical, the best
rate counts, and with both kinds the effect goes through both,
`magical * (physical * effect / 100) / 100`. It is applied after the base
damage and before the critical and the variance. A skill carries its
`attribute_effects`; an actor's normal attack, the `attribute_set` of the
weapons in its first two slots; an enemy's attack, nothing. An enemy's rank
is its `attribute_ranks`; an actor's, its own, one step up if any armor it
wears carries the attribute. Since 2026-10-18 a skill with
`affect_attr_defence` —Heat Barrier, Cold Barrier— shifts its target's rank
in each attribute it carries one step for the rest of the battle, more
resistant on an ally and weaker on an enemy, never more than one step
either way, with a roll against the skill's hit chance per attribute and no
roll at the bound (`Game_Battler::ShiftAttributeRate`); the damage reads the
shifted rank. A healing skill from the menu goes through the same, on the
actor it heals, with no shift. See `rm2k-snes/battle/src/attributes.rs` in
the-advocate, the model, and `runtime/battle_attributes.s`, the console.
What is not the same:

* **A skill with a physical attribute does not ask for the weapon.** In
  RM2k an actor cannot use one —it is grayed out— unless a weapon it holds
  carries that attribute (`Game_Actor::IsSkillUsable`), except for those
  with `affect_attr_defence`; here it is offered as any other. In *A
  Blurred Line* that is Saldra's Trip, which her Bent Staff does not carry
  (Martial), and AG7-17's ArmLaser and TillTheyGlow with the ArmCannon,
  which carries no Ray.
* **An effect past 32,767 before the variance is held there**: the console
  multiplies in thirty-two bits and keeps sixteen. No game comes near: the
  damage's cap is 999.

## How it applies

The tables are the converter's (`attributes` in `emit.rs` of
rm2k-snes/mapdata), outside bank 0, and only for the attributes a carried
skill or weapon can carry; the scratch is the zero page's. The shifts take
far RAM, a byte per combatant and attribute, only for the attributes some
carried skill can shift. The tests are `tests/rpgmaker/test_battle_attributes.py`
and `tests/rpgmaker/test_battle_attribute_shift.py` of A Blurred Line,
against the model and against easyrpg's arithmetic. The tasks are
[`elemental-attributes-are-not-applied-in-battle`](https://github.com/lautarodragan/the-advocate-work/blob/main/tasks/elemental-attributes-are-not-applied-in-battle.md)
and
[`skills-that-shift-an-attribute-rank-do-not-shift-it`](https://github.com/lautarodragan/the-advocate-work/blob/main/tasks/skills-that-shift-an-attribute-rank-do-not-shift-it.md).
