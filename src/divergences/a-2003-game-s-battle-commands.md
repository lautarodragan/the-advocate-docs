# A 2003 game's battle commands

## What

An RPG Maker 2003 game is fought with the console's battle, which is RPG
Maker 2000's: one window with five commands —attack, skill, defend, item
and escape—, in that order, for every actor. RPG Maker 2000 names them with
its terms, `command_attack`, `command_skill` —or the actor's own
`skill_name`—, `command_defend`, `command_item` and `battle_escape`. RPG
Maker 2003's battle never shows those terms: it names each actor's
commands with the database's **battle commands**, of which every actor
carries a list —and a class too, which `ChangeClass` copies—. So in a 2003
database the 2000 terms hold whatever the editor left in them: Zounds'
`command_attack` and `command_defend` are the editor's Japanese defaults,
which the console wrote as `?UOE'` and `-hOEa` until 2026-10-20.

Since that day, in a 2003 game, each of the five commands takes the name of
the first command **of the actor's own list** that does the same, by its
`type` —attack, skill, defense, item, escape—; one the actor's list lacks
takes the database's first command of that type, and one the database
lacks too, the 2000 term. Zounds' Kazar uses items with his own
"Tirar-Usar", and Unamë with "Objeto". See `command_names` in
`rm2k-snes/battle/src/rules.rs`, and `actor_command_txt` in
`rm2k-snes/mapdata/src/emit.rs` and in `runtime/battle_skills.s`, in
the-advocate.

## What RM2k3 does

easyrpg's `Game_Actor::GetBattleCommands` walks the actor's list —skipping
`0`, the row command, and `-1`, an empty slot— and
`Scene_Battle_Rpg2k3::GetBattleCommandNames` writes each one's `name`, in
the list's order, one window per actor. What each does is its type's: an
item command, whatever it is called, opens the item window; a **subskill**
command lists only the skills of its own set; a **special** command does
nothing by itself and is what the troops' pages with the `command_actor`
condition react to. The 2000 `battle_escape` is still written, in the
party's window —fight, auto, escape—, which the gauge and alternative
battles open with cancel.

## What is not there

* **The actor's order and its presence**: the console's window is always
  the five, in the 2000 order —see
  [Escape is the fifth command](escape-is-the-fifth-command.md)—. Zounds' Unamë and Kazar carry their item
  command before Defender, and Bandurion carries no escape at all.
* **The row command**, `0` in the list —which easyrpg does not show
  either— and **subskill** commands: none shows. Zounds' "Robar" and "Juglarismo" are subskills, but no actor or
  class carries them.
* **Special commands**: the console's escape stands in for the one a game
  escapes with —Zounds' "Escapar", 9, whose troop pages count the turns
  spent trying—, with the 2000 escape's chance. See
  [`the-2003-battle-commands-callcommonevent-and-forceflee`](https://github.com/lautarodragan/the-advocate-work/blob/main/tasks/the-2003-battle-commands-callcommonevent-and-forceflee.md).
* **A list changed while playing**: the names are the database's, read when
  the cartridge is built. `ChangeClass` and `ChangeBattleCommands` do not
  change them.

All four are in the draft
[`a-2003-actor-s-command-window-is-the-2000-one`](https://github.com/lautarodragan/the-advocate-work/blob/main/tasks/a-2003-actor-s-command-window-is-the-2000-one.md).
The lines the 2000 battle writes for each hit are another divergence:
[A 2003 game's battle writes the 2000 lines](a-2003-game-s-battle-writes-the-2000-lines.md).
