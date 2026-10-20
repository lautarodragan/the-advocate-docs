# The pause menu and its options

The pause menu is RM2k's `Scene_Menu` at 256x224: the command column on the
left, the party on the right and the gold at the bottom, over black, in the
same tile bank and palette as the save screen. B opens it when the hero is
standing still, and the game clock does not run while it is up. The code is
`runtime/menu.s` in the-advocate, and the geometry and the blocks come from
`rm2k-snes-screens`, whose `render` is the reference.

## The options

The column is a list of **rows**, and each row shows a **command**: the
runtime builds the list each time the menu opens —`menu_rows_build`, in
`runtime/custom_menu.s`—, so an option can come and go without the others
knowing. In order:

| Row | Command | What it opens |
|---|---|---|
| RM2k's first three | Items, Skills, Equipment | the item screen, and the skills and equipment screens after choosing whom on the menu itself; grey with the party empty |
| one per custom menu | the game's custom menus | a full-screen menu the game describes as data, while its condition holds —the console in A Blurred Line; see [the custom menu](../research/2026-10-04-the-custom-menu.md)— |
| RM2k's last two | Save, End Game | the slot screen, grey while `ChangeSaveAccess` forbids it; the End Game screen |
| the last one | Debug | the debug screen, only with `[debug] debug_screen` (below) |

RM2k's five are commands 0 to 4, the custom menus 5 and on, and the debug
screen the one after the last custom menu. The labels are drawn by
`rm2k-snes-screens` into the menu's bank, two blocks each —the normal one
and the grey one—, by command; the row they go on is the runtime's. Every
option that opens a full screen —the custom menus, the skills screen, the
debug screen— takes the whole VRAM, and the map comes back as from a big
picture when the menu closes: `CM_DIRTY`, and `big_back_far`.

## The debug screen

RPG_RT's F9 —`Scene_Debug`, only in a test play—, which the console has no
key for: the switches and the variables of the database, turned on and off
and changed by hand. It is `runtime/debug_screen.s` and
`rm2k-snes/screens/src/debug_screen.rs`, and it is what RPG_RT 2000 shows,
measured under Wine:

* the ranges on the left, ten rows of `S[0001-0010]`, and the list on the
  right, ten rows of `0001: Name` with `[ON]` or `[OFF]`, or a variable's
  value right-aligned;
* in the ranges, up and down go round the page's ten, left and right flip
  pages of ten ranges, round, and after the switches' pages come the
  variables' —`V[0001-0010]`—; A goes into the list and B leaves;
* in the list, up and down go round its rows, A turns a switch on or off,
  left and right change a variable by one and L and R by ten, and B goes
  back to the ranges.

**A change takes effect when the map comes back**, as in RPG_RT: a switch
or a variable that changes asks for the pages' refresh, and the pure
parallel common events run again when the menu closes. Where it differs
from RPG_RT is in [What the debug screen does not
model](../divergences/what-the-debug-screen-does-not-model.md).

The names go in a block of their own in a far bank, by reference, cut to
what fits; the code is in `FAR3`. A cartridge that does not ask for the
screen carries none of it: the code is inside an `.if DEBUG_SCREEN`, and the
converter writes only `DEBUG_SCREEN = 0`.

## The `[debug]` knobs

The project file's `[debug]` table holds what a port turns on while it is
being debugged and off for a clean delivery. Every key is required, like
the rest of the file:

* **`debug_map`**: a map of the port's own, with a door to each key point of
  the game, where "Begin the Story" starts —see A Blurred Line's project
  file—. It has to be the only new map of the manual patches.
* **`enemy_hp`**: the battle writes the enemies' HP, which RM2k never shows;
  the converter's `--enemy-hp`.
* **`debug_screen`**: the pause menu's last option opens the debug screen,
  above; the converter's `--debug-screen`. Off, the ROM is the same byte for
  byte as one built before the screen existed.

`rm2k-snes-cartridges` reads the three and passes the flags to every
cartridge of the project.
