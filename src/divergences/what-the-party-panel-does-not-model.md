# What the party panel does not model

## What

The party's rows on the pause menu and on the actor choice —the one an
item or a healing skill used from the menu opens— are RM2k's
`Window_MenuStatus` and `Window_ActorTarget` at 256 pixels wide and 224
tall, where RPG_RT has 320 by 240. Each row is 56 lines, the face on the
left and the text beside it, and four rows are the whole screen. What does
not fit is laid out differently:

* **One column of text, in four lines.** RPG_RT puts the HP to the right
  of the level and the SP to the right of the experience; in 104 pixels
  there is room for one column, so the HP go on a line of their own under
  the level and its state.
* **The SP are only on the actor choice**, on its fourth line, under the
  HP and in the same columns: `Window_ActorTarget` has no experience, so
  that line is free. On the pause menu the fourth line is the experience,
  and **a fifth one does not fit with four actors**: a line of the font is
  ten rows of ink with its shadow, five of them need some 55 of the row's
  56, and the experience already takes 120 of the 128 sprites. The menu
  shows neither the SP nor the actor's title; the skills screen shows the
  SP of the one it is opened for.
* **The fourth line is sprites**, not tiles: the experience on the menu
  and the SP on the choice. The other three lines are on the cell grid,
  two cells each, and only an object can sit eleven lines under another.
* **The bar reaches the screen's last line** on the actor choice's fourth
  row, over the bottom of the party window's frame: the row's text goes
  down to there, and the bar covers it with a line of red over it and two
  under it. RPG_RT's window keeps its frame below the last row.

## Why

The console's screen is 256 by 224 and the font is RM2k's, six pixels
wide and twelve tall: RPG_RT's layout needs 320 by 240. The measurements
are in the task.

## How it applies

The geometry is `MENU_BAR`, `MENU_EXP_X` and `MENU_SP_X` in `layout.rs` of
`rm2k-snes-screens` in the-advocate, whose `render` draws both screens; the
console's side is `menu_party_paint` in `runtime/menu.s` and
`menu_exp_paint` and `menu_sp_build` in `runtime/menu_experience.s`. The
tests are `tests/rpgmaker/test_menu_heal_drawing.py` of A Blurred Line, on
the debug map's full party. The task is
[`sp-in-the-menu-and-the-item-target-picker`](https://github.com/lautarodragan/the-advocate-work/blob/main/tasks/sp-in-the-menu-and-the-item-target-picker.md).
