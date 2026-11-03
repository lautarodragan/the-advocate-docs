# What the party panel does not model

## What

The party's rows on the pause menu and on the actor choice —the one an
item or a healing skill used from the menu opens— are RM2k's
`Window_MenuStatus` and `Window_ActorTarget` at 256 pixels wide and 224
tall, where RPG_RT has 320 by 240. Each row has the face on the left and
the text beside it. What does not fit is laid out differently:

* **One column of text on the pause menu.** RPG_RT puts the HP to the
  right of the level and the SP to the right of the experience; in 104
  pixels there is room for one column, so the HP go on a line of their own
  under the level and its state, and the SP on another under the HP, in
  the same columns.
* **The pause menu shows three actors, not four, and scrolls.** Its rows
  have five lines —name, level, HP, SP and experience—, 72 lines each, and
  four of them would be 288. RPG_RT's window shows the four at once; here
  the panel shows three, and when the bar choosing whom a command is for
  goes past them it moves a whole row, as a `Window_Selectable` moves its
  top row. Under the third row goes **the System's arrow**: the down one if
  someone is below, the up one if someone is above. It does not blink, as
  RPG_RT's does every twenty frames, and both go at the bottom, centred
  on the panel: at the top the up one would cover the first name.
* **The actor choice shows the four**, the whole width under the item's
  line, with three lines each and the SP beside the HP: see [The actor
  choice's layout](the-actor-choice-layout.md).
* **The menu's last line is sprites**, not tiles: the experience. The
  other lines are on the cell grid, two cells each, and only an object can
  sit eleven lines under another. The menu's SP are tiles: in sprites, with
  the experience, three rows would need 144 objects, and there are 128.
* **The menu does not show the actor's title.** The skills screen shows
  the SP of the one it is opened for.
* **A condition is written in one shade**, where the row says "Normal"
  otherwise: RPG_RT draws its name with the System colour's gradient, some
  eight shades from top to bottom. The rows' colours are in palette 0,
  which in some games is full, and the faces take the others: so each
  condition colour gets **one entry of palette 1**, the disabled commands',
  the mean of its gradient weighted by how much of the font each row
  covers, and each row of the letters goes to that entry or to the
  palette's nearest, whichever is nearer. When the free entries run out,
  a colour gets none and its rows take the nearest ones: they go first to
  the dead condition, then to the ones that outlive a battle, then by
  number. The skills screen's status line has a palette free, and there
  the condition keeps its gradient.
* **A condition's name is cut at the screen's edge**, 62 pixels from where
  it starts. RPG_RT writes it whole, over the HP if it is long.

## Why

The console's screen is 256 by 224 and the font is RM2k's, six pixels
wide and twelve tall: RPG_RT's layout needs 320 by 240. On the menu, with
four rows of 56 lines, a fifth line of ink does not fit —five lines need
some 55 of the row's 56— nor in sprites —the experience took 120 of the
128—. Of the ways out —drop the state, drop the experience, the 512-pixel
mode— taro chose the scroll on 2026-11-01. The measurements are in the
tasks.

## How it applies

The condition is `MENU_STATE_COL` and `StateInks` in `slots.rs` of
`rm2k-snes-screens`, and `runtime/menu_states.s` on the console, with
`tests/rpgmaker/test_menu_conditions.py` of A Blurred Line and the task
[`the-menu-shows-the-partys-conditions`](https://github.com/lautarodragan/the-advocate-work/blob/main/tasks/the-menu-shows-the-partys-conditions.md).
The geometry is `MENU_LIST_ROWS`, `MENU_SHOWN`, `MENU_ARROW_Y`,
`MENU_BAR`, `TARGET_BAR`, `MENU_EXP_X` and `MENU_SP_X` in `layout.rs` of
`rm2k-snes-screens` in the-advocate, whose `render` draws both screens;
the console's side is `runtime/menu_list.s` —the rows, painted once each
when the menu opens and kept in WRAM, so a scroll is one vblank's
upload, and the SP line in cells, `menu_sp_cells`— and `menu_exp_build`
in `runtime/menu_experience.s`. The tests are
`tests/rpgmaker/test_menu_party_scroll.py` and
`tests/rpgmaker/test_menu_heal_drawing.py` of A Blurred Line, on the debug
map's full party. The tasks are
[`sp-in-the-menu-and-the-item-target-picker`](https://github.com/lautarodragan/the-advocate-work/blob/main/tasks/sp-in-the-menu-and-the-item-target-picker.md)
and
[`the-main-menu-scrolls-its-party-for-sp`](https://github.com/lautarodragan/the-advocate-work/blob/main/tasks/the-main-menu-scrolls-its-party-for-sp.md).
