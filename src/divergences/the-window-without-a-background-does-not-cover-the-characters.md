# The window without a background does not cover the characters

## What

`MessageOptions` without a background leaves the text over the map, with
its shadow, where the window would go. In the ROM that has two differences
from RM2k, and both come from the same thing: the window is **BG2**, the
same layer as the one above the map, and mode 3 does not have a third.

* **The characters go over the text.** In RM2k the window is above
  everything —`Priority_Window` is 130, and the characters go from 30 to
  70—. Here the text is BG2 with priority 1, and the hero and the events are
  objects with priority 2, which in mode 3 go in front. With a background it
  does not show because HDMA hides the objects on the window's lines
  —`mask_*`, in `message_place.s` in the-advocate—; without a background they
  cannot be hidden, because they have to be seen.
* **The layer above the map is not visible on the lines the window draws
  on.** On those lines BG2 shows the window's tilemap —the plane without a
  frame, with the cells that are not text left blank—, so roofs, treetops
  and everything the chipset puts on top disappear from them while the
  message lasts, **across the whole width of the screen**, beside the text
  too. The lower layer and the characters are visible. In a map whose
  panorama goes in mode 3 the panorama is that BG2, so it is the panorama
  that disappears from those lines: Vampires Dawn's intro, which is all
  panorama, asks "Skip Intro ?" without a background, and the ROM shows it
  over a black band where RM2k shows the picture.

**Which lines the window draws on**, out of its 80: they go in bands —the
frame's top row, its four lines of text and its bottom row—, and the
window takes BG2 only on its own bands:

* **a line of text**, sixteen lines: the two rows of cells the composer
  writes per line, glyph and shadow included. A line the box does not
  carry, or an empty one, gives the map back;
* on a **choice**, every option, empty or not, so that the window does not
  change as the bar moves; and an `InputNumber`'s digits, which are one
  more line;
* with a **face**, the four lines, which the face covers;
* and the **bottom row**, eight lines, where the arrow of a message
  waiting for a key goes —RM2k draws it with the window's opacity at zero
  too—, unless the page cannot show it: a choice, the digits, or a `\^`
  closing it. The top row, never.

So a message of one line takes 24 lines from the layer above, and one of
three takes 56. Until 2026-10-18 it was always the 80: in A Blurred Line,
Core and Manta's talk on the island of their memory took the tops of the
trees under the window's empty lines.

**On a painted map the first one is the other way round.** There the
painting is BG1 in mode 1, and on the lines the window draws on BG1 and BG2
trade places so the letters go in front of the cells of the painting that
are in front of the hero —see [A painted map keeps only what the screen
shows](../internals/vram.md#a-painted-map-keeps-only-what-the-screen-shows)—:
the text is BG1 with priority 1, which in mode 1 goes in front of the
objects of priority 2, so the hero and the events on his layer or below go
**under** the text, as in RM2k. An event "above the hero" is priority 3
there, and it still goes over it. The second one stays: the edges of the
roofs that go exact in BG2 are not visible on those lines.

## The limit: beside the text

The cells to the left and right of the text, on a line that has text,
cannot be given back, and that is a property of the PPU and not of how far
this went:

* **BG2 has one tilemap and one scroll per scanline.** `BG2SC`,
  `BG2HOFS` and `BG2VOFS` are written by HDMA in the horizontal blank, so a
  scanline is the window's from edge to edge or the map's from edge to
  edge: there is no changing the tilemap in the middle of a line.
* **The PPU's windows only hide.** Window 1 or 2 masking BG2 outside the
  text's columns would show what is *below* BG2 there —the lower layer,
  BG1— and not the layer above, which is another tilemap that, on that
  line, is not being read at all.
* **The layer above has nowhere else to go.** BG1 is the lower layer, mode
  3 has no third background, and objects cannot carry it: the PPU draws 34
  sprite tiles a line among all of them, and the characters already use
  them.

So what a line of text costs is the width of the screen, not the length of
the line: "CORE" takes the layer above off the same sixteen lines as a
full line.

## Measured

Against the original under Wine, on 2026-10-04, with the page from
`window-middle-no-background` placed in a copy of the game: the text lands in
the same place and with the same shadow, and the only difference on screen
is that the hero covers "Ready". The captures from that comparison were
kept outside the repositories, for review.

ABL asks for the window without a background in more than a third of its
`MessageOptions`, almost always in cutscenes over the black screen or a
picture, where neither difference shows. Where it does show —Core and
Manta's memory, over the island's trees—, ABL's
`test_the_layer_above_shows_off_the_text` looks at a message of one line:
on the window's lines without text the screen is the same as with the
window closed.

## When it stops being enough

In a game that talks without a background over a map with roofs or trees
beside the text, or with characters walking through it. Fixing it means
drawing the text with something other than BG2: with objects it does not
fit —a line of text is 30 tiles wide and the PPU draws 34 per line among
all of them—, and mode 3 has no more layers.
