# What a skill's result lines do not model

## What

Since 2026-10-20 a skill in battle —or a special item that casts one—
writes, under its using lines, what it did on each target, as RM2k's 2000
battle does: one message per line, each in the place of the one before,
the using lines staying above —`Scene_Battle_Rpg2k`'s
`ProcessBattleActionDamage` and `ProcessBattleActionParamEffects`, and
`ProcessBattleActionFailure` when it did nothing—. The lines are those of
`game_message_terms.cpp` with the database's terms:

    Junkbot 12 health lost.          the damage, `GetDamagedMessage`
    Junkbot avoids taking damage.    a damage of zero, `GetUndamagedMessage`
    Talan HP 40 Recovered            `GetHpRecoveredMessage`, what was recovered
    Talan SP 10 Recovered            and the SP, or `<SP> 10<parameter_decrease>`
    Droid Attack 5                   a parameter, `GetParameterChangeMessage`
    Droid was not affected.          the failure, by the skill's `failure_message`

The model says the same lines —`results` in `rm2k-snes-battle`'s answer—
and A Blurred Line's `test_battle_skill_results.py`, in the-advocate-ports,
holds the console to them. See `ba_res_add` in `runtime/battle_skills.s` and
`result` in `ba_message`, `runtime/battle_message.s`, in the-advocate. A medicine's
recovery line writes the points with `health_points` and `spirit_points`
since the same day, as RM2k does, and not with the party window's short
`hp_short` and `sp_short` —they differ in The Book of Three—.

What is not the same yet:

* **No line for a condition or an attribute rank**: no skill gives or takes
  conditions in this battle —see
  [What battle conditions do not model](what-battle-conditions-do-not-model.md)—,
  and the attributes' names are not in the cartridge, so a skill that only
  shifts a rank writes nothing under its using lines. It still counts as
  having done something: it is not a failure.
* **No line for the death**: RM2k writes the dead state's message when the
  damage kills —`ProcessBattleActionDeath`—. The attack does not write it
  either.
* **No absorbing**: a skill that absorbs HP or SP does what one that does
  not would, and says it that way.
* **No sound with the damage**: RM2k plays the ally's or the enemy's
  damage sound with the damage line; here a skill plays only the item
  sound, with its using lines.
* **The HP move before the line**: RM2k applies the damage after writing
  it, so the party's window changes with the line; here every target's
  HP has moved when the using lines come up.
* **The window shows the first four rows**: RM2k scrolls its message
  window to the last line. A skill whose two using lines take more than
  three rows leaves its result out.
* **What comes after `vExecute` returns is still rolled and applied**:
  RM2k stops once the damage kills, and when a skill against enemies
  missed the HP or the SP it touches. The lines stop there here too, but the
  rolls and the parameters after them do not —the draft
  [`a-skill-rolls-past-where-vexecute-returns`](https://github.com/lautarodragan/the-advocate-work/blob/main/tasks/a-skill-rolls-past-where-vexecute-returns.md)—.

## Measured

Against easyrpg-player's source, on 2026-10-20: `Scene_Battle_Rpg2k`'s
`ProcessBattleAction*`, `Game_BattleAlgorithm::Skill::vExecute` and
`GetStartMessage`, and `BattleMessage` in `game_message_terms.cpp`. A
Blurred Line's `test_battle_skill_results.py` has Talan's MinorFire hit a
Junkbot, Kersh throw a Carbon Battery and give a Cloudy Potion, and Kersh's
Agency Badge lower a droid's attack, defense and agility, against the
model; its golden `battle-skill-damage` is MinorFire's line.

## When it stops being enough

When a game's skills give conditions, which is when their lines would
matter: Vampires Dawn's and The Book of Three's. And when one's using lines
are long enough to push the result out of the window.
