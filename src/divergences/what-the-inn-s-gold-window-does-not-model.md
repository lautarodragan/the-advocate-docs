# What the inn's gold window does not model

## What

A `ShowInn` with a price shows its greeting with RM2k's gold window next to
it —`Window_Gold`, 88x32 at the screen's right edge, at the top, or at the
bottom when the message is at the top— with the party's gold and the gold
term, and greys its "Yes" out when the party cannot pay: A on it does
nothing and the choice stays open; "No", and cancelling, still close it.
That is easyrpg's `CommandShowInn`, with `SetShowGoldWindow(true)` and
`PushChoice(accept, can_afford)`, and since 2026-10-19 the console does it.
What it does differently:

* **The greeting is its own box and the choice another**, as with every
  `ShowMessage` before a `ShowChoice` on the console: the greeting waits for
  a key, and the gold window closes with it and opens again with the
  choice, a few frames later. In RM2k the greeting and its two options are
  one window, and the gold window stays.
* **The grey is the window palette's nearest colours to System colour 3**,
  row by row of the glyph: in A Blurred Line, lighter at the top and darker
  below than the original's gradient. The window's letters come composed
  with their colour, and its palette has no entry to spare —the maps'
  backgrounds start right after it, see [The window takes thirteen colors
  from every background](the-window-takes-thirteen-colors-from-every-background.md)—,
  so a greyed letter has its glyph's pixels moved to entries the window
  already has.
* **A on the greyed "Yes" plays nothing**: RM2k plays the System's buzzer,
  and the console plays no System sound anywhere.
* **The gold window needs room in the map's sprites**: it is 44 8x8
  objects, after the message arrow's two tiles past the sprite bank and in
  the sprite palette nobody takes —see [The window without a background
  does not cover the characters](the-window-without-a-background-does-not-cover-the-characters.md#the-arrow-as-objects)—,
  because mode 3 has no other layer for it. A map without that room shows
  the greeting without it, and the build lists the map as
  `inn-no-gold-window`: in A Blurred Line, map 200, whose eight sprite
  palettes are taken. Under a big picture it does not show either.
* **The gold window is drawn when the box opens**: if a parallel event
  changes the gold while the greeting or the choice is up, the window keeps
  the number it opened with, until the next box.

## Why

The gold window and the grey are the inn's debts that were left when
`ShowInn` first compiled, and an inn without them lets the player try to
pay for a night they cannot afford. The objects are the only place left on
a map in mode 3, and the message arrow had already shown where they fit.

## How it applies

`compile_inn` in rm2k-snes-events puts two phases of `OP_INN` before the
greeting —3, the gold window and "Yes" greyed, and 4, which a gold branch
reaches only with the price paid for, the grey off—; `gold` in
rm2k-snes-screens draws the empty window with the gold term and the
grey's entries; `arrow_obj` in rm2k-snes-mapdata says which maps have room;
and `runtime/gold_window.s`, in the-advocate, composes the number, puts the
objects up and refuses the greyed option. A Blurred Line's
`tests/rpgmaker/test_inn.py` and `test_inn_golden.py` cover it, the goldens
compared with the 2015 RPG_RT under Wine.
