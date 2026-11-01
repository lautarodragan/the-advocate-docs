# What battle conditions do not model

## What

Since 2026-10-12 the battle keeps RM2k's conditions —the database's
states— on the enemies, as `Game_Battler` does: a troop page's
`ChangeMonsterCondition` gives one or takes it away, and from then on it
counts its turns, wears off on its own when `hold_turn` and
`auto_release_prob` say so, is released by a hit with `release_by_damage`,
restricts what its holder does —it does nothing, or attacks a random one of
its own side or of the other— and halves the attack, defense, spirit or
agility it says, and lowers its holder's hit chance with
`reduce_hit_ratio`. Its two messages come out: `message_affected` when its
holder's turn comes, and `message_recovery` when it goes. *A Blurred Line*
asks for two, both from Talan's systems: `Control` (22), from
`ControlMachines`, and `BarkWall` (23), from `Inscribe`. See
`runtime/battle_states.s` in the-advocate, and `Held` in
`rm2k-snes/battle/src/simulate.rs`, the model that rolls the same dice in
the same order.

Since 2026-11-01 the actors bring theirs into the battle and take them
out —see "Outside battle"—, and **a skill gives and takes them** on either
side, as `Skill::vExecute`: after the parameters, for each condition of
its `state_effects`, one against enemies says `message_already` if the
target holds it, and otherwise rolls the skill's hit and then the
condition's odds on the target —`GetStateProbability`, the rate of the
target's rank, and on an actor what its armor leaves of it— and gives it,
saying `message_actor` or `message_enemy`; one on its own side rolls the
hit and takes it away, saying `message_recovery`. A special item casts its
skill, so its conditions come with it. And **each time its holder acts**,
right after the releases, a condition changes its HP and SP by
`hp_change_val` plus `hp_change_max` percent of the maximum
—`ApplyConditions`—, a loss never below 1 HP. What is not the same:

* **A weapon's `state_set` gives nothing**: an attack does not roll its
  `state_chance`. Vampires Dawn has twenty such weapons, ABL three.
* **An item that only cures conditions is not in the battle's list**, and
  a medicine's `state_set` takes nothing away in battle.
* **An actor that cannot act is still asked for its command**, and then
  does nothing —or attacks, with Berserk and Chaos— when its turn comes,
  as RM2k; RM2k skips it in the command menu. Its status line says
  `normal_status` or the dead one.
* **The enemies' AI does not look at conditions**: a skill on its own side
  that only takes conditions away weighs as if it did nothing —see
  [What the enemies' AI does not model](what-the-enemies-ai-does-not-model.md)—.
* **A condition does not seal skills** (`restrict_skill`,
  `restrict_magic`): Silence in the default database, ABL's Stone.
* **A skill that gives death kills without the dead one's message** of its
  own beyond the condition's line, and the dead one is never taken away by
  a skill: no target of a skill is fallen here.
* **Taking the dead condition away from an enemy does not compile**: in
  RM2k it gets up with 1 HP, and the scene does not draw again one it
  erased. No game asks for it. Adding it kills, without a message or a
  sound, as RM2k —Book of Three does it on troop pages—.
* **Four at once per enemy.** A fifth is not given. No game comes near.
* **What a hit releases is said in a window of its own**, after the hit's,
  and only the first if one hit releases two; RM2k writes it as a third
  line under the damage, one per condition.
* **The holder does not flash white** when its turn comes with a
  condition's message —`ProcessBattleActionBegin`'s `Flash`—: the message
  comes out, in a window of its own before the action, as in RM2k.
* **A physical skill's hit chance does not look at conditions**, as it does
  not look at agility either: it is the skill's `hit`. RM2k lowers it with
  `reduce_hit_ratio` and always hits one that cannot act when the skill's
  `failure_message` is 3. A normal attack does both.

The randomness is the battle's own, as everywhere else in it: see
[What the enemies' AI does not model](what-the-enemies-ai-does-not-model.md).

## Outside battle

Since 2026-11-01 the actors hold the database's conditions as
`Game_Actor`'s `states`, in four places like an enemy's —see
`runtime/party_states.s` in the-advocate—: `ChangeCondition` gives and
takes them on an actor or the party, `IsActorConditioned` asks for them,
`FullHeal` and the inn clear them, a fall purges them as death's priority
says, they travel in the save, the battle takes them in and gives them
back, and its end drops the `battle_only` ones and the persistent ones that
wear off on their own. Each pad step, `Game_Party::ApplyStateDamage`: the
HP and SP a condition changes every so many of the party's steps, never
below 1 HP, with the red flash. What is not the same:

* **Equipment that gives or prevents a condition** —RM2k's
  `PermanentStates`, and an armor's resistance— is not looked at. No game
  of the nine equips one that gives one.
* **The step's changes go actor by actor**, each one's conditions in
  order; RM2k goes condition by condition, each one's holders in order.
  The HP come out the same unless one actor holds a condition that heals
  walking and another that hurts on the same step: no game has one that
  heals.
* **The menus do not show them** yet: the main menu's status line and the
  status screen say nothing, and a medicine or a skill that only cures a
  condition cures nothing on the map.

## Measured

On 2026-10-12, against the model, turn by turn —the HP of everyone, each
enemy's conditions with their count, its parameters and the conditions'
messages—, in the two battles where the game gives them: the Scrapped
Droid and the GiantRat with `Control` (troop 111), and the Lumbering Giant
with `BarkWall` (troop 128).

On 2026-11-01, against the model, in Zounds: Kazar throws a Veneno at the
Pez Shark, with the battle's randomness set so the poison takes hold, and
the shark's HP after it acts —its poison's 5 % and 1 included— and its
conditions with their count are the model's.

## How to reproduce it

`tests/rpgmaker/test_battle_conditions_golden.py` in the ABL port
(the-advocate-ports/abl), with the cases `battle-control` and
`battle-inscribe`. And `tests/test_zounds_veneno.py` in the ports,
with the case `battle-veneno` of Zounds.

## When it stops being enough

When a game's battles rely on skills, weapons or items that give
conditions, or on poison in battle: the party's half outside battle is
there, the battle's is [`the-partys-conditions`](https://github.com/lautarodragan/the-advocate-work/blob/main/tasks/the-partys-conditions.md).
