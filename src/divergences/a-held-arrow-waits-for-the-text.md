# A held arrow waits for the text

## What

RM2k repeats a held arrow every four frames, whatever the screen is
doing —`Input::IsRepeated`, easyrpg's `src/input.cpp`—. The ROM repeats it
the same way, `arrows_repeat` in `runtime/control.s` in the-advocate,
**except on the screens whose text is composed** —items, equipment,
skills, the shop, the battle's lists—: there the repeat fires only once the
screen shows what it wrote last, and waits otherwise. A press is never held
back, only a repeat.

What looks different: holding an arrow over a long list moves the cursor at
RM2k's pace inside a page and slower when the list scrolls, because each
step waits for its text. Measured on 2026-10-18 on ABL's item screen with
thirty items: four or five frames per step inside the page, six to ten per
step that scrolls; on the skills screen, eight to twelve.

## Why

Those screens compose their text into tiles while the frame waits for the
vblank, and the tilemap switches only when all of it is composed, so the
screen goes from one state to the next in one frame —`text_layer.s`—. A
step that scrolls a list takes longer than four frames to compose, so with
RM2k's repeat the screen wrote over a screen that was never shown, the
text stood still for a second or two, and when the pool of tiles ran out
the tilemap switched halfway, with the rows that came in showing their
neighbour's item. taro saw it on 2026-10-18; see
[the task](https://github.com/lautarodragan/the-advocate-work/blob/main/tasks/scrolling-a-list-shows-the-last-row-repeated-for-a-moment.md).
Waiting is `tl_pace`, in `runtime/text_layer.s`.

## Measured

`test_holding_an_arrow_shows_every_row_s_item`, in ABL's
`tests/rpgmaker/test_menu_items_drawing.py`, holds down over thirty items
and up again, and requires every frame to show a whole list and a
description the cursor was on together. If the composition gets faster,
the pace gets closer to RM2k's on its own: nothing here counts frames.
