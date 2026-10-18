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

## The composed glyphs carry palette entries, not colours

Each row of a System's text is a single colour —the gradient is vertical—
and the shadow is flat, so the precomposed glyphs say which palette entry
each row takes, and the colour is the palette's. The converter gives the
text the same entries in every System a cartridge switches to, so the
message window's glyph tables are carried once for all of them, and a
`ChangeSystemGraphics` is a palette and a few tiles; and the screens' text
lays every ink out the same way —row k is entry 1 + k, the shadow 13—, so
one set of tables paints every colour of the items, equipment, shop and
battle.

- **Saves**: ROM —the eight tables, one per phase, are most of the font's
  weight, and are not carried per System nor per ink— and the CPU of sampling the System's colours per
  pixel. The converter refuses a System whose text rows are not one colour
  each.
- **Where**: `text_rows`, `common_rows` and `font_prerender` in
  `rm2k-snes/screens/src/message_window.rs`; `ink_colors` in
  `rm2k-snes/screens/src/items.rs`; `compose_char` in `runtime/messages.s`,
  `tl_glyph` in `runtime/text_layer.s`, and `runtime/window_system.s`.
- **More**: the header of `runtime/window_system.s` and the comment on
  `tl_glyph`.

## A message is composed a few letters a frame, a line a vblank

The message window's text is composed by the main loop a handful of
letters per frame into a buffer of one line, and that line goes up in the
next vblank as one DMA; the window turns on when the box is whole. No
frame carries more than its share of composing, and no vblank more than a
line.

- **Saves**: frame time and vblank time while a box is built, and bank 0's
  RAM —one line's buffer, not the window's—.
- **Costs**: a box takes some frames to appear, more the more letters it
  has; RM2k opens its window with an animation.
- **Where**: `MSG_CHARS`, `message_tick` and `upload_line` in
  `runtime/messages.s`.
- **More**: the comment on `MSG_CHARS`, with what was measured.

## A wait in the middle of a box draws its line again

A box with `\.`, `\|` or `\!` is cut at the wait: the window shows what
there is, the wait runs, and the box is composed again from the start of
the line that was cut, over what was already there. Keeping the half-drawn
line while blanking the ones below would need a second line buffer.

- **Saves**: bank 0's RAM —a second buffer of a line—.
- **Costs**: a few frames of the composer after each wait.
- **Where**: `msg_wait_mark`, `msg_hold_tick` and `msg_line_start` in
  `runtime/message_waits.s`.
- **More**: the header of `runtime/message_waits.s`.

## The party panel is one block for every row and every actor

The pause menu's and the actor choice's party rows share one fixed block
—the level and HP terms, the status— whoever is in the row; the name, the
level and the maximum go over it, the name composed into tiles of its own
and the numbers with the bank's glyphs.

- **Saves**: VRAM and ROM —the menu's tile bank does not grow with the
  number of actors—. The slot screen does the same with its leader's name
  and level.
- **Where**: `menu_party_paint` in `runtime/menu.s`; `slot_party_paint`
  and "the text that depends on the actor" in `runtime/slots.s`; `tl_strip`
  and `tl_slot_name` in `runtime/text_layer.s`.
- **More**: the comment on `menu_party_paint`.

## A renamed actor's name is one byte, and the others are baked

Every name a `ChangeHeroName` can give is a literal of the game, so the
converter writes them all as texts, and what changes at run time is one
byte per actor some `ChangeHeroName` renames: which of its names it
carries. An actor nobody renames keeps the database's name, which a
message's `\N[n]` gets baked into its text, letter by letter.

- **Saves**: RAM and SRAM —no name buffer per actor—, and the CPU of
  writing a name the box can already carry. A game that renames no one
  carries no byte and no table.
- **Where**: `actor_name_txt`, `actor_name_row_now`, `change_name` and
  `msg_name_mark` in `runtime/actor_names.s`; `with_names` and
  `OP_CHANGE_NAME` in the event compiler.
- **More**: [What names in messages do not model](../../divergences/what-names-in-messages-do-not-model.md).

## End Game's cursor is a palette

End Game's two windows come drawn whole, and its two pictures —the bar on
"Yes" and on "No"— share the tiles and the tilemap: each pair of colours a
pixel takes in the two pictures is a palette entry, so moving the cursor is
writing the other picture's palette.

- **Saves**: VRAM —no second tilemap and no tiles of a second bar— and the
  vblank of moving the cursor, sixteen colours.
- **Where**: `end_game.rs` in `rm2k-snes/screens/src/`; `ending_palette`
  in `runtime/end_game.s`.
- **More**: the header of `end_game.rs`; what happens when those pairs do
  not fit is [lossy](../lossy/tiles.md#a-window-with-a-gradient-is-fused-until-it-fits).

## The shop's party: its frames when the vblank has room, its grey a palette

The shop's party walks as objects. Its OAM is written once, when the shop
opens; a member that cannot equip the item under the cursor is greyed by
giving its objects its grey palette, not other tiles; who can equip it is
worked out only when the item or the layer changes; and a member's next
frame goes up only if the vblank's line leaves room after the text and the
windows, or the next frame.

- **Saves**: vblank time, the tightest budget there is, and VRAM.
- **Costs**: a member can step a frame or two late after a key.
- **Where**: `ti_party_open`, `ti_party_compute` and `ti_party_vblank`,
  with its `room` and `TI_PARTY_VBLANK`, in `runtime/shop_party.s`.
- **More**: the header of `runtime/shop_party.s`; [The shop's party
  window](../../divergences/the-shops-party-window.md).
