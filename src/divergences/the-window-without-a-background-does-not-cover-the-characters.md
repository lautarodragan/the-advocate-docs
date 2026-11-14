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
  cannot be hidden, because they have to be seen. The arrow of a message
  waiting for a key does go over them, in a map with room for it: see
  [The arrow, as objects](#the-arrow-as-objects).
* **The layer above the map, on the lines the window draws on, is the
  window's copy of it.** On those lines BG2 shows the window's tilemap
  —the plane without a frame—, and BG2 is the layer above too. Since
  2026-11-14 the plane carries the layer there: the cells without letters
  take the layer's own tilemap entries, and the cells with letters are
  composed from the layer's pixels with the glyph over them, so roofs,
  treetops and fences show beside the text and under it, as in RM2k —see
  [The layer above, carried](#the-layer-above-carried)—. **Only with the
  camera still**, which is when it is on the 8-pixel grid the window's
  cells are on: a window that opens while the camera moves takes the
  layer off its lines across the whole screen, as before that day, and
  one under which the camera moves keeps the layer as it was when it
  opened. So does a window with a face, a choice or a number's digits,
  and one in a game whose boxes type colours (`\c[n]`).

  Where the window does not carry the layer the old look holds: on those
  lines BG2 shows the window's cells, so the layer above disappears from
  them **across the whole width of the screen**, beside the text too, and
  the lower layer and the characters are visible. In a map whose panorama
  goes in mode 3 the panorama is that BG2, so it is the panorama that
  disappears from those lines: Vampires Dawn's intro, which is all
  panorama, asks "Skip Intro ?" without a background, and the ROM shows it
  over a black band where RM2k shows the picture. And where the lower
  layer has nothing under the layer above —the dark edge of a cave, a wall
  drawn only on top— what is left is the backdrop, so those cells turn
  **black** on the window's lines: Zounds' signs on maps 86 and 96,
  measured on 2026-11-02.

  **Only what a character can pass under is in that layer**, since
  2026-11-02: a tile above the hero over nothing —the black those cells
  turned— on a cell no sprite can ever cover —no character can stand
  beside it or reach it, by walking, by a route, a jump or a door— is
  drawn into the background, where it looks the same and the window does
  not take it. The cave's ceiling far from the floor
  stays; its edge by the floor, and anything a character can walk behind,
  still goes. Which cells, and what the analysis cannot prove and so keeps
  above, is in [Maps](../internals/maps.md#what-goes-in-the-layer-above).

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
  too—, **only in a map without room to draw the arrow as objects** —see
  below— and unless the page cannot show it: a choice, the digits, or a
  `\^` closing it. The top row, never.

So a message of one line takes 16 lines from the layer above, and one of
three takes 48; in a map without room for the arrow's objects, 24 and 56.
Until 2026-10-18 it was always the 80: in A Blurred Line, Core and Manta's
talk on the island of their memory took the tops of the trees under the
window's empty lines, and later that day the trees just under the window
got back the top row of tiles the arrow's band still took.

## The arrow, as objects

In a window without its background the arrow is **two 8x8 objects**, and
the bottom row's eight lines are the map's but for the arrow's own pixels.
It is where RM2k puts it: over everything but the timer —`Priority_Window`
is 130 and `Priority_Timer` 140—, so priority 3, in front of both layers,
and the first objects after the timer's in OAM, which is what puts it over
the characters too: between two objects the PPU draws the one with the
lower index. It blinks with the same count as in the cells, twenty frames
up and twenty off.

It needs, per map, **two tiles past the sprite bank** that the layout's
sprite tiles still reach, and **a whole sprite palette nobody takes**: the
top one the pictures leave, above the characters' and the battle
animation's box. Into them go the window's own arrow tiles and the window's
colours, so the System is not redrawn nor loses a colour, and the screen's
tint does not reach it, which is what RM2k does with the window. The
converter decides it —`arrow_obj` in rm2k-snes-mapdata, a word per map in
`arrow_objs`— and the console uploads both when the window appears
—`arrow_obj_choose` in `message_arrow.s`, in the-advocate—.

**A map without that room keeps the arrow in the window's cells**, and the
bottom row's band with it. The converter's log says which, map by map:
`the message arrow stays in the window's cells`, with why —the bank
reaching the layout's sprite tiles, or the eight palettes taken—. In A
Blurred Line it is the eight palettes almost always: the busy maps, with
seven palettes of characters and a battle animation's box or a picture.
The window keeps the cells too while a big picture is up, which has the
window's tilemap elsewhere and its tiles over the sprite bank.

**What the PPU does not draw**: on a line with more than 32 objects, the
first 32 in OAM order, so never the arrow; but with more than 34 tiles on
a line —time over— it fetches tiles from the last of those 32 back to the
first and drops what does not fit, so the arrow, early in OAM, is the
first to lose them. Each character crossing the arrow's lines is four
tiles: up to eight fit with the arrow, and with nine the line was already
over without it.

A window **with** its background keeps the arrow in its cells: it takes
BG2 on its eighty lines anyway, and its mask hides the objects on them.

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

## The layer above, carried

A line of text the window draws on is two rows of its plane, and the
console fills them from the map's layer above when the line starts —see
`runtime/message_layer.s` in the-advocate—: it reads the layer's tilemap
entries for the screen cells the two rows cover, out of VRAM in a vblank,
and the cells the text area leaves without a tile of the layer keep the
window's own. A cell of the text area that has one is read too, its 8x8
pixels into the composer's buffer, and the letters are composed over them
as over the window's transparent. The cell shows the layer's entry until a
letter inks it, and from then the composed tile. The window's bottom row,
the arrow's band in a map that keeps the arrow in the window's cells,
takes the layer's entries too, and the arrow's two cells give them back
when it blinks off.

**The camera at rest is always on the 8-pixel grid**: `follow_camera`
puts the centre of the hero's cell —8 mod 16— at the middle of the screen,
128 and 112, and its clamps are 0 and the map's size in cells times 16
minus the screen's. So with the camera still the screen's cells are the
map's, and the window's cells land on them without the text moving a
pixel from where RM2k draws it. A window that opens with the camera
between two cells keeps the old look; carrying the layer there would move
the text up to seven pixels, or the composer would draw it shifted, three
cell rows a line.

**The colours.** The composer's glyphs carry the window's palette
entries, and the composed cell is drawn in a palette of the layer's, so
that palette has to hold the text's colours at the window's entries —the
glyph's rows' and the shadow's, five in A Blurred Line—, and in entries no
pixel of the layer uses: under a tint, an entry the layer and the text
shared would be tinted for both or for neither. The converter decides it
map by map, in a game with a `MessageOptions` without the background
—`layer_text` in rm2k-snes-mapdata—:

* if the layer's palettes have room, its colours move off the text's
  entries, and nothing changes;
* if not, and the background leaves the BG palettes under the layer's, the
  text goes in **copies** of the layer's palettes there, used only by the
  cells with letters; the layer's colours nearest the text's are moved to
  the text's entries, so inside those cells they show as the text's —taro's
  rule of 2026-10-19: the text keeps its exact colours and the nearest of
  the background give way—;
* if neither, the layer's least visible colours are merged into their
  nearest until its palettes have room, **only if they are near** —no two
  more than some two steps of 31 apart, and none losing its tint—: the
  layer above loses those shades on that map, with or without a message.
  Otherwise the map keeps the old look, and the converter's log says why.

The text's entries are written again after every tint with the window's
colours, untinted, so the letters are not toned and the layer is, as RM2k
tones the map and not the window.

## The limit: beside the text

Where the window does not carry the layer —see above—, the cells to the
left and right of the text, on a line that has text, cannot be given back
by the PPU alone, and why is what led to carrying it:

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
window closed, and on the bottom row too but for the arrow's sixteen
columns, where it is up some frames and not others. The arrow as objects
lands on the same pixels as it did in the cells, measured against the
cells on 2026-10-18.

The layer carried on the lines with text, since 2026-11-14:
`test_message_layer_above.py`, in ABL, looks at taro's two scenes of the
memory —map 5's "I can tell." over a treetop, map 6's first box over the
fence at the screen's left edge—, and at map 5's under the memory's sepia
tone: with the box up, every pixel of its lines is the map's as it was
before it opened or one of the text's untinted greys, and the bottom row
is the map's but for the arrow's two cells. Before that day 243 pixels of
the tree and 1237 of the fence were neither.

## When it stops being enough

In a game that talks without a background over a map with roofs or trees
beside the text **while the camera moves** —a parallel route walking the
hero, a pan—, or with characters walking through the text: they still go
over the letters, because the text is BG2 and they are objects in front of
it. Drawing the text with something other than BG2 does not fit: with
objects a line of text is 30 tiles wide and the PPU draws 34 per line
among all of them, and mode 3 has no more layers.

Zounds is such a game in another way: its signs —a picture of a wooden
board with the text over it, without a background— stand in caves whose
dark edges are the layer above, and RM2k draws the board under the text,
which the console leaves out of those maps for lack of VRAM; drawn, it
would hide the bands, but as objects it would also go over the text. Its
boxes type colours, so its windows do not carry the layer.

The camera moving under a message is its own task, in the wishlist:
[`a-transparent-message-under-a-moving-camera`](https://github.com/lautarodragan/the-advocate-work/blob/main/tasks/a-transparent-message-under-a-moving-camera.md).
