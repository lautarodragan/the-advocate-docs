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

**It runs in the console's 512-wide mode**, mode 5, since 2026-10-20
—taro's: at 256 wide a third of A Blurred Line's switch names were cut—.
The letters are the font's same pixels, each half as wide on screen, so a
line holds twice as many; the windows are drawn at 256 wide like every
other screen's and go up with each pixel twice, so they look as they did.
In mode 5 a tilemap word is a cell 16 pixels wide, two tiles side by side:

* **the text is BG1, at 4bpp**, with the text layer in its wide mode
  —`tl_hires` in `runtime/text_layer.s`—: a column of the layer is four
  tiles, a *quad*, where elsewhere it is a pair, and its inks are the
  items' white and grey, moved to palettes 2 and 3;
* **the windows are BG2, at 2bpp**: eight palettes of three colours in
  the first 32 entries of the colour memory, which Adeleine's
  `reduce_colors` fills from the System —one with more colours, or more
  than three in a cell, loses some; A Blurred Line's has three—;
* **the main screen and the subscreen show both layers**, which is what
  mode 5 needs to show every pixel of the 512; no objects.

When B leaves, mode 3 and no subscreen again, which is what the pause menu
draws in. The VRAM, in [VRAM](vram.md#the-debug-screen-is-512-wide).

The converter measures the screen with that width: the ranges' window is
as wide as its widest label, and a name is cut only if it does not fit in
what is left —none of A Blurred Line's—. The names go in a block of their
own in a far bank, by reference; the code is in `FAR3`. A cartridge that
does not ask for the screen carries none of it: the code is inside an `.if
DEBUG_SCREEN`, and the converter writes only `DEBUG_SCREEN = 0`.

**A row of the list is the screen's whole row** for the text layer —the
range's label beside it is the same record—, so a step that changes the
list would compose the ranges again. The layer keeps a record's head when
it did not change and the rest starts further right —`tl_split`—, so
within a page the ranges are never composed again, and a step asks the
pool only for the list's columns.

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
