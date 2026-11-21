# ShowWindow: a window in a picture's place

## What

**RPG Maker 2000 PLUS has a `ShowWindow`**: a window like a picture,
numbered and put up and taken down by the events, whose pixels are not a
file. The console draws its frame and background from the System in use
and composes its text with the message window's font, at run time. It was
made for the menus games build out of `ShowPicture`s —an option lit is a
picture, the same option unlit another— that the console has no room to
show on most maps.

RM2k's editor has no new commands and the game has to keep running in
RPG_RT, so it is written as a **`Comment`** whose first line is a
directive, and its lines the window's text:

```text
@window <id> <x> <y> <w> <h> [plain]
\c[1]Inventory
Keys
```

and `@window_erase <id>`. **When the command right after the comment is a
`ShowPicture` of picture `<id>`, the window stands in for that picture**:
the console compiles the window and not the picture, and an
`ErasePicture` of `<id>` takes the window down too. RPG_RT skips the
comment and shows the picture, so a game forked this way runs there as it
always did. A comment with no picture after it is a window alone, which
RPG_RT never draws.

`x`, `y`, `w` and `h` are pixels of **the console's 256x224 screen**, in
multiples of 8: RPG_RT does not draw the window, so there is no RM2k
position to translate. A framed window's text starts 8 pixels in; a
`plain` one —the System's background without its frame— at its corner.
Each line is 16 pixels, as a message's, with RM2k's escapes: `\c[n]`,
`\V[n]`, `\N[n]`. A higher id is drawn over a lower one, as a picture
with a higher number is.

## Why

taro, on 2026-11-20, after studying Desert Nightmare's room: a common
event's pictures are, almost always, custom systems —a menu, a grid, a
description—, and the experiment worth doing is replacing them one to one
with a "`ShowMessage` with a parametrisable x, y, w and h", on the 8-pixel
grid and with two to four of the System's colours, so they show inside the
maps without taking the screen. The pictures are read once as text —by a
person, or by a model with a prompt the port keeps—.

**A comment in front of the picture, and not a table in the project**
that says "picture `Optionen2` is this window", because the fork is then
a reviewed patch of the game's own data, as the ports' other forks are,
and not a rule the build applies; and because the comment is also how a
game written for PLUS asks for a window with no picture behind it.

## How it applies

- **Stage one: the message window's bank, with no message open.** A
  window takes the message window's own 32x32 tilemap and its 240 text
  tiles, as a `ShowChoice` does, so it fits on every map and costs no
  sprite tile. It cannot be on screen with a message, as none of RM2k's
  own menus can: when a box opens the windows are taken off and drawn again
  when it closes. A game that opens a box over its own menu of pictures
  —Desert Nightmare's "You have no items."— sees its menu go away while
  the box is up. Windows beside a message —in objects, as the inn's gold
  window— are the second stage, if a game asks for it.
- **The text tiles are fixed per window id for the whole game**: each id
  gets the most that one `@window` of it takes —its lines packed, two rows
  of cells each—, and every id of the game together has to fit 239 tiles
  —the 240th is the windows' background—. The build says by how much when
  they do not. Two windows of the same id are never up together, as two
  pictures of the same number are not, so a grid of six squares serves
  the inventory, the keys and the notes with the same tiles. Until
  2026-11-20 an id had room for the widest text of each of its lines, and
  Desert Nightmare's keys and notes, its inventory's windows with other
  text, did not fit beside it.
- **A big picture covers the windows under its number**, as a picture
  covers the lower ones in RM2k: the big ones are a background layer, under
  the windows', so after a big picture's `ShowPicture` the windows with a
  lower id are left off the screen, and its `ErasePicture` puts them back.
  Desert Nightmare's notes and maps are big pictures over its menu's
  windows.
- **The layer above stays beside a window narrower than the screen**
  (taro's requirement, 2026-10-19): on the lines a window covers, the
  window's tilemap carries the layer above's entries in every cell no
  window covers, read out of VRAM, as a message without its background
  carries them on its text lines —"The layer above, carried" in
  [the transparent window's divergence](../divergences/the-window-without-a-background-does-not-cover-the-characters.md)—.
  With the camera on the 8-pixel grid, which is where it is at rest.
- **Two colours exactly, more approximately**: colour 0 and the first
  other colour the windows use take the boxes' ink; a third goes with the
  System's nearest, as in a box.
- **A port may need an image-to-text step**, a prompt kept beside its
  patches —Desert Nightmare's `windows/prompt.md`—, done once and
  reviewed, and only if it is trivial: copied into a chat with the
  pictures, and run on a 20 USD plan without taking a significant part of
  its five-hour limit (taro, 2026-11-20).
- How it is built is in `runtime/show_window.s` and `windows.rs` of the
  event compiler, in the-advocate; the task is
  [`show-window-a-window-command-of-our-own`](https://github.com/lautarodragan/the-advocate-work/blob/main/tasks/show-window-a-window-command-of-our-own.md).
