# Lossy: events

## Translucent events are baked

The blend of a page's "Trans" —RM2k draws the event at 160 out of 255— is
computed in the converter against the layer below its cell and stays in
the graphic: no colour math, no dithering, no separate palettes. It can be
done exactly because the translucent pages A Blurred Line has are all
static —`move_type` 0 and no route of their own—: if neither the object
nor what is behind it moves, the blend always gives the same result.

The other two ways are not enough there. Colour math applies to every
sprite in palettes 4 to 7 at once, and was tried and left out —see [Tried
and left out](../tried-and-left-out.md#colour-math-for-translucent-events)—.
Dithering, which is what games of the era did, does not simulate
transparency over a dark background: the fires on ABL's map 9 sit on
almost pure black, and dithering punches holes in the flame.

- **Loses**: nothing is seen through them; over an animated tile the
  blend is right for one frame; two over different backgrounds are two
  graphics. The CharSets in the project's `[graphics] opaque` —the save
  point— go opaque instead.
- **Where**: `decorate_translucent` in `rm2k-snes/convert/src/game.rs`,
  which names the graphic; `rm2k-snes/wally/src/sheets.rs`, which draws it
  with `translucent` from `rm2k-render`.
- **More**: [Translucent events end up opaque](../../divergences/translucent-events-end-up-opaque.md),
  the divergence, with what was measured and when it stops being enough.

## Decorative events become background

With `[graphics] decorative_events`, an event that is only a drawing is
painted into the background and stops being an event.

- **Loses**: its animation follows the ChipSet water's rhythm and phase;
  a translucency that does not fit gets dithered; blocking moves to the
  grid.
- **Saves**: sprite tiles, rows and slots.
- **Where**: `rm2k-snes/wally/src/decor.rs`.
- **More**: [Baked decorative events](../../divergences/baked-decorative-events.md).

## A still wall left out is the map's passability

A map over the console's `EV_MAX` events leaves some out, and an invisible
wall that never stops being one —every page on the hero's layer, none that
walks, one without conditions, nobody names it— goes first: its cell is
closed in the map's passability, so the hero is still stopped there.

- **Loses**: an event below or above the hero that walks into the cell is
  stopped too, which RM2k does not do; a drawn one loses its drawing.
- **Saves**: a row of the RAM's event tables per wall.
- **Where**: `still_wall` in `rm2k-snes/convert/src/trim.rs`; the
  request's `walls`, which `rm2k-snes/wally/src/layers.rs` closes.
- **More**: [Events that do not fit in the RAM](../../divergences/events-that-do-not-fit-in-the-ram.md).

## Still events that animate past the sprite bank

A still event that animates changes pose on the fly, so it needs a VRAM
slot that still events showing its pose cannot share; the ones with the
same graphic, pose and speed share one among themselves, since they always
show the same column. A map whose slots don't fit gives up whole looks
—a graphic and pose, all its speeds— until they do, before anything else
is lost.

- **Loses**: on those maps, the still events of the looks given up stand
  still.
- **Saves**: a sprite slot per speed of each look given up.
- **Where**: `moves` in Wally, `rm2k-snes/wally/src/moves.rs`, and
  `fewer_still_animations` in `rm2k-snes/wally/src/map.rs`.
- **More**: [Still events that animate past the sprite bank](../../divergences/still-events-that-animate-past-the-sprite-bank.md).

## Still events that animate share one phase

A still event that runs through its walk cycle has no byte of its own to
count in, so its column comes from the game tick, the same for all of
them, and not from a counter that starts when the map creates it.

- **Loses**: which of the four steps each event starts on.
- **Saves**: RAM —a counter per event— and, because they are all in step,
  the VRAM slot that the ones with the same graphic, pose and speed share.
- **Where**: `animate_still` in `runtime/animated_stills.s`.
- **More**: [The phase of still events that animate](../../divergences/the-phase-of-still-events-that-animate.md).

## A common event declared a counter is one opcode

A common event that draws a variable as two digit pictures can be
declared a counter in the project's `[events] counters`, and the compiler
replaces each call to it with `OP_COUNTER` —the variable and where—, which
draws the number with the timer's glyphs, clamped to 0..99, the tens
erased under ten. On 2026-09-30, A Blurred Line's common event 9 pasted
at each of its 31 calls was 1,345 bytes of `ConditionalBranch` chains
each.

- **Loses**: whatever else the common event draws —the "Fortitude" label
  is not drawn—, and the digits are the timer's, not the game's pictures.
  That the common event means a counter is taken on trust from the
  project's declaration; only where it draws is checked.
- **Saves**: ROM and WRAM —the body pasted at every call— and the CPU of
  running it.
- **Where**: `compile_counter` in `rm2k-snes/events/src/compile.rs`;
  `op_counter` in `runtime/interpreter_timer.s`, `counter_set` and
  `counter_oam` in `runtime/counter.s`.
- **More**: the header of `runtime/counter.s`.

## A common event declared a menu is a screen of the engine

A common event that builds a menu out of pictures and waits for keys can be
declared a custom menu in the project's `[menus]`, with a file that writes
what each of its pages says; the compiler replaces the common event with
`OP_MENU_OPEN` —and turns its switch off, as the game's own body did when
the menu closed—, and the runtime draws it as a full-screen screen of the
engine, with the System's windows and the text layer.

- **Loses**: the game's own pictures and layout —it looks like the pause
  menu, not like the game's menu—. What the menu shows is taken on trust
  from the project's file, written by hand; nothing checks it against the
  common event.
- **Saves**: the pictures —A Blurred Line's menu was one of 61— with their
  VRAM, palettes and the CPU of moving them, and the common event's body.
- **Where**: `OP_MENU_OPEN` in `rm2k-snes/events/src/commons.rs`;
  `custom_menu.rs` in `rm2k-snes/screens/`; `runtime/custom_menu.s`.
- **More**: the header of `runtime/custom_menu.s`;
  [The custom menu](../../research/2026-10-04-the-custom-menu.md).
