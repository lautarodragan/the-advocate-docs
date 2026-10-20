# What pictures as sprites do not model

## What

The `ShowPicture` pictures that go in sprites —see
`runtime/picture_far.s` and `rm2k-snes/images/src/show_picture.rs` in the-advocate— do
not do everything they do in RM2k:

* **Transparency comes in three steps**: at 76 % or more it is not visible,
  from 25 % to 75 % it blends half and half with what is underneath —the
  console's color math, which blends at 50 % or adds, and nothing in
  between—, and below that it is opaque. The smoke on 207 (40 %) comes out
  at 50 %, and the shadow on 311 appears all at once halfway through its
  one-second fade, already at 50 %. While there is a blended one, the
  characters the map has in sprite palettes 4 and 5 are blended too, and so
  is the battle animation's box if it lands there: the hardware blends
  palettes 4 to 7 entirely. It is the `pic_blend` experiment, in
  `runtime/picture.s`, and it comes out with its commit.
* **`MovePicture` does not enlarge**: it changes tone, transparency and
  place —since 2026-10-16; until then not the place—, each picture with its
  own ramp. One that asks for another zoom does not compile and is accepted
  without being done. One that goes with the map stays where it is, as in
  RM2k 2000; RM2k 2003 would move it.
* **Sixteen numbers at a time on a map**, and what goes in is what the map's
  characters leave: the pictures share the sprite palettes they get from
  the top, and the 512 object tiles. A map whose pictures do not fit loses
  all of them, not the last one: Zounds' menu on map 136, two faces of 180x59
  and a bar as wide as the screen, needs 512 tiles. Until 2026-10-16 it was
  two at a time, one palette each.
* **Pictures that share a palette lose their least used colours** to the
  nearest one the palette has, two steps of 31 away at most —the blends a
  digit's scaling leaves at its edges—. A number the map tones does not
  share: the console tones a picture by toning its palette.
* **They count against the line's objects and slivers**: the console shows
  32 objects on a line and 34 slivers of eight pixels. A block of a picture
  with four of its sixteen cells drawn or fewer goes as 8x8 objects, and
  since 2026-10-20 so does one with up to eight in no more than two of its
  four rows —a frame's edges, a line of text—, unless they are two whole
  rows side by side; a 32x32 costs four slivers on each of its 32 lines,
  drawn or not. Pictures over many characters, or several wide ones on the
  same lines, can still lose some of their pieces: what a line holds is
  272 pixels of objects. Zounds' skill screen goes in only without its
  selector's fill, which its port leaves out —see its
  `patches/manual/README.md` in the-advocate-ports—.
* **No rotation or waving, no enlarging, no saturation**, and nothing wider
  than 256 —nor taller than 128 if it is placed by a constant with the
  screen, which is then a big one; one placed by variables, or with the map,
  goes up to 256 tall since 2026-10-20—: those belong to the BG1 plane.
* **Fifteen colors**, those of one sprite palette, counted by use: gradients
  get banded, and a colour few pixels use can go —the title of Zounds'
  record-player bar comes out the bar's blue—.
* **They go above the message window**: object priority 3, which is the one
  from before. In RM2k the window covers the pictures.
* **The ones tied to the screen are scaled to 256x224, the map's are not**:
  on the console the map is not shrunk, so a picture attached to it keeps
  its pixels and only its zoom is applied. Nor are a port's own pictures
  drawn for the console, which its project names in `[graphics]
  console_pictures`: see [A port's own files replace the game's, by
  name](../decisions/a-ports-own-files-replace-the-games-by-name.md).
* **The screen shake** moves the map's pictures —`screen_x` is subtracted
  from them, as from the characters— but not the screen's.
* **A common event's pictures show only where their set fits beside the
  map**: a common event compiled without a map —automatic, parallel, or a
  body one of those calls— shows its small pictures from a table the
  cartridge shares, and their tiles go up past the loaded map's bank when
  one is shown, if the map leaves room for all that the common event and
  the bodies it calls show; the converter's log says on how many maps each
  one fits. One set at a time: showing another common event's takes
  down what the first had up. And they take their slots from the top, so a
  map that shows sixteen numbers between its own and theirs loses some.
  See [The common events' small pictures go up when
  shown](../internals/vram.md#the-common-events-small-pictures-go-up-when-shown).
* **A page that goes on after a `Teleport`** and shows a picture uses the
  table of the map it was compiled in: the destination's may be another.
  Measured over the game's maps: only one page does it, the one on 288, and
  its two pictures are large ones.

## Measured

On the test game's `test_pictures.py`, on 2026-10-16: six pictures on the
meadow at once —three digits in one palette and one tile each, a panel, a
second digit over it and a bar—, two of them sliding in sixty frames at the
same time, and one toned without toning the others.

On the `release` cartridge, on 2026-10-01: the sign on 4, the one on 81, the
Fortitude's label on 95 and the shadow on 311 are drawn. The shadow lands in
the same place on the map as in RPG_RT under Wine: its box gives
(39,59)-(140,179) there and (40,60)-(138,178) here, in map pixels. The
`ABLrec` on 45 is not drawn because the map uses seven character palettes
and the battle animation's box the eighth.

On Zounds' `release`, on 2026-10-20, with the-advocate's branch
`carpintero-ce-pictures`: the record player's bar —common event 36— slides
down over the debug room and comes back after the debug screen, in
`test_zounds_common_pictures.py` of the ports. The permit fits on 53 of the
189 maps, the notes on 96, the record player on 128 and the music bar on
145; the pick-axe screen —400 tiles and seven palettes, most of the tiles
the 320x80 box under its messages— and the notebook —704 tiles— on none.

## When it stops being enough

When 207 gets into the cartridge —the smoke is the only translucent one
that is looked at up close— or a common event's screen that no map leaves
room for: Zounds' pick-axe and notebook, which would need the characters to
give up the plane while they are up. See [More pictures at
once](../research/2026-10-16-more-pictures-at-once.md).

## As an optimization

What it saves, and where it lives in the code: [Pictures in sprites: scaled, fifteen colours](../optimizations/lossy/pictures-and-animations.md#pictures-in-sprites-scaled-fifteen-colours), in Optimizations.
