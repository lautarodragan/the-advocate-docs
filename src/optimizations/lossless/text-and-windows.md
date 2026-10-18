# Lossless: text and windows

## A repeated box is stored once

`OP_MESSAGE` carries an offset into one block of text for the whole
cartridge, not the text itself, so a box the game repeats —whole boxes
repeat often— is stored once: the content is the key.

- **Saves**: ROM in the messages.
- **Where**: `rm2k-snes/events/src/messages.rs`.
- **More**: [Events](../../internals/events.md).

## DTE in the messages

The byte values the text never uses each stand for a pair of letters, one
level only, so a box is still read on its own.

- **Saves**: ROM in the messages.
- **Where**: `rm2k-snes/events/src/messages.rs`; the ROM side,
  `runtime/message_letter.s`.
- **More**: [Events](../../internals/events.md),
  [The whole game without MSU-1](../../ports/abl/research/2026-10-01-the-whole-game-without-msu-1.md).

## Glyphs pre-rendered in the cartridge

The variable-width font is stored already shifted to every phase of a
tile, so composing a letter is copying, not shifting pixels.

- **Saves**: CPU while text is written —it costs ROM instead—.
- **Where**: `font_prerender` in `rm2k-snes/screens/src/message_window.rs`;
  `compose_char` in `runtime/messages.s`.
- **More**: [Events](../../internals/events.md).

## The message window's text area from one line

When every line of the message window's text area comes out the same, the
cartridge carries one line and `prime_line` copies it under each.

- **Saves**: ROM.
- **Where**: `prime_line` in `runtime/messages.s`.
- **More**: [The windows' background is the project's, by System](../../decisions/the-windows-background-is-the-projects-by-system.md).

## One tilemap, two planes, placed by HDMA

The message window's tilemap holds two planes of ten rows —with frame and
without—, and HDMA says which one is seen and on which lines.

- **Saves**: VRAM —one 2 KB tilemap for every position and style of the
  window—.
- **Where**: `runtime/message_place.s`; `build_hdma` and `msg_hdma_build`.
- **More**: [The VRAM split](../../internals/vram.md).

## A face borrows the window's text cells and, by HDMA, palette 1

A message's face is drawn into the text cells of the window that the text
leaves free beside it, and its fifteen colours are written into BG palette
1 by HDMA only on the window's lines, the map's coming back below it.

- **Saves**: VRAM and CGRAM —a face takes none beyond the window's—.
- **Where**: `runtime/message_face.s`; `msg_face_vblank`.
- **More**: [What faces do not model](../../divergences/what-faces-do-not-model.md),
  [The CGRAM split](../../internals/cgram.md#a-messages-face-borrows-palette-1-on-the-windows-lines),
  [The VRAM split](../../internals/vram.md#a-messages-face-uses-the-windows-own-cells).

## The screens' text takes tiles only where it has ink

The items', equipment's, shop's and battle's text is composed row by row
into pairs of tiles from a pool, one pair per column that gets ink, and a
row is typeset again only when it changed. A row that says what another
one said takes that one's tiles, which each pair counts, so a list that
scrolls by one row composes one row.

- **Saves**: VRAM —a list of ten rows fits— and CPU.
- **Where**: `runtime/text_layer.s`, `TL_FREE` and `TL_REFS`.
- **More**: the header of `runtime/text_layer.s`.

## The screens' text is typeset while the frame waits for the vblank

The text layer typesets its rows in the frame's idle time, called from
`wait_vblank` until the line it is on reaches a limit, and the tilemap
goes up once everything is typeset, in a vblank that carries no tiles, so
a screen switches in one frame. The row is composed in the 2 KB buffer
the battle animation uses on the map, which no screen needs.

- **Saves**: frame time on the screens, and far RAM.
- **Costs**: the text arrives some frames after the cursor.
- **Where**: `tl_idle` in `runtime/text_layer.s`, from `wait_vblank` in
  `runtime/boot.s`; `scr_upload` in `runtime/text_screens.s`; `TL_STAGE`.
- **More**: the header of `runtime/text_layer.s`.

## The window's arrow and the shop's windows go up only when they change

The message window's arrow writes its cells in the vblank only on the
frame it appears or goes away, and the shop's windows and bar only when
the mode, the greeting, the cursor or the list changed, not every frame.

- **Saves**: vblank time, the tightest budget there is.
- **Where**: `msg_arrow` in `runtime/message_arrow.s`; `ti_windows` in
  `runtime/shop.s`.
- **More**: the comments of those routines.
