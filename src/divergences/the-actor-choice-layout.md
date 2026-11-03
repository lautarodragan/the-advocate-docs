# The actor choice's layout

## What

The actor choice —the screen an item or a healing skill used from the menu
opens, RM2k's `Scene_ActorTarget`— puts **the item's line on top and the
party below it, the whole width**, where RPG_RT puts the item on the left
and the party on the right:

* **On top, one System window, 32 lines tall**, with the item's or the
  skill's name on the left and, on the right, "Owned Items" or the skill's
  cost term with the count. RPG_RT has two windows of 136 by 32 on the
  left, `Window_Help` with the name and `Window_TargetStatus` with the
  count under it.
* **Below, the four party rows, 48 lines each, without a window**, like the
  pause menu's panel. RPG_RT's `Window_ActorTarget` is a window of 184 by
  240 on the right, with ten lines between rows.
* **Each row is RPG_RT's three lines** —the name; the level and the
  condition; the HP— with the **SP beside the HP**, where RPG_RT puts them
  under the HP, on the condition's line.
* **The term is right-aligned to the count**, and the name has the room
  before it, which depends on how long the game's term is; RPG_RT's help
  window gives it 120 pixels at its six-pixel font. A name that does not
  fit is cut where it reaches the room, and the converter says which with
  a WARNING.
* **The name keeps its ExFont icon**, as RPG_RT's `Window_Help` draws it,
  and its letters the font lacks go as their plain neighbours, as in the
  item list: "Poción" is "Pocion".

The bar is RPG_RT's: from four pixels left of the text, past the face,
which it does not cover —`Window_ActorTarget::UpdateCursorRect`, `48 + 4`—,
for one row, for the whole party, or fixed on the user's.

## Why

At 256 pixels wide the two sides do not fit. Until 2026-11-02 the left
column was 104 pixels, eleven letters: a medicine's name went on two lines
—Zounds' "Poción Verde" as "Poci?n" over "Verde"—, and the party's window
started at 104 so that ABL's faces, which leave twelve transparent pixels
on their left, did not cover it. Zounds' faces fill their 48, and were
drawn over the windows' frames. taro's layout, from that day: the item's
line on top, in one row, and the party list the whole width, faces
included. It applies to every game, not only to the ones with faces that
fill them: all nine ports have faces, and one layout is one screen to draw
and to test.

The party goes without a window because four rows of a face's height are
192 lines, which is what the 224 leave under the item's line: a window's
two frames would be sixteen more, and RPG_RT's layout is 240 tall. Four is
RM2k's most, so nothing scrolls.

## How it applies

The geometry is `TARGET_WINDOWS`, `TARGET_PARTY_ROW`, `TARGET_ACTOR_COL`,
`TARGET_SP_COL` and `TARGET_BAR` in `layout.rs` of `rm2k-snes-screens` in
the-advocate, whose `render::target` draws the screen; the console's side
is `runtime/actor_choice.s` —the window, the name and the count— and
`menu_party_paint` and `target_sp_cells` in `runtime/menu_list.s`, the
rows: the pause menu's block, moved to the left edge. The tests are
`tests/rpgmaker/test_menu_heal_drawing.py` of A Blurred Line, against the
render, and the golden of `tests/test_zounds_item_target.py` in the ports.
The task is
[`zounds-the-item-target-picker-does-not-fit`](https://github.com/lautarodragan/the-advocate-work/blob/main/tasks/zounds-the-item-target-picker-does-not-fit.md).
