# Nights with a tint of their own (2026-11-19)

> **Measured on 2026-11-19** by Hurón, on Desert Nightmare's `release`
> built from the-advocate-ports `aa850b4` (before) and the branch
> `huron-dn-nights` at `6d16d18` (with the patches), with the tools of
> the-advocate `4f38ea03`, in snes9x through the ports' harness, against
> EasyRPG Player on the original game and on a copy with the same patches.
> It is a snapshot: the numbers are from that day and this file is not
> updated. The task is
> [`desert-nightmare-nights-with-a-tint-of-their-own`](https://github.com/lautarodragan/the-advocate-work/blob/main/tasks/desert-nightmare-nights-with-a-tint-of-their-own.md).

taro, looking at the towns painted, translated: "the before and after are
much brighter than EasyRPG's. I see the same in 59, 61, 62, 66 […] if it is
not a tint per se but an image, I prefer using map patches to use a normal
RPG Maker tint, and analyse how we can solve the lights creatively, with
their acceptable limitations." This is why the nights were day, the
patches that tint them, and what each way of lighting a tinted night
would give, measured.

## Summary

* **The game has no night tint.** A night is two pictures the size of the
  map: `Lichteffekt_*_Nacht`, a near-black mask (9, 9, 9) with the lamps'
  cones in a gradient, at transparency 20 —35 or 45 indoors—, and on the
  streets `Nacht_*` over it, opaque, a transparent canvas with only the
  lit windows and bulbs. The night panoramas are the day ones with a few
  lamps changed.
* **On the maps that scroll the pictures are placed by a variable**, and
  the converter does not draw those: the page reads an anchor event's
  screen position into variables 14 and 15 and shows them there. Eight
  nights showed as day —32, 59, 61, 62, 66, 68, 69, 70—, 8 to 12 of 31
  levels off at 15 bits on the streets. On the one-screen nights (60, 63,
  64, 65, 67, 72, 121 and the interiors) the place is a constant and both
  pictures are already drawn into the map, lights and all.
* **Map 2's darker capture was the capture's timing**: EasyRPG was halfway
  through the map's own fade-in from black. Map 2 is day.
* **The patches**: a fork of each of the eight maps, the mask's
  `ShowPicture` turned into the `TintScreen` that comes closest at 15 bits
  —(25, 25, 25, 85) on the streets—, the windows' picture removed and the
  tint set back to neutral before every door. The console against EasyRPG
  running the same forks: **0.05 to 0.26** of 31; against the original,
  **0.6 to 1.4**, from 2.5 to 12.4 before. What is left is the cones and
  the windows.
* **The lights**: the cheapest faithful way for the map is to draw the
  cones and windows into the cells the tint misses, in palettes the tint
  skips (0.1 to 0.5 over the cones); the only one that also lights a
  character walking into a cone is colour math adding a 2-bit layer over
  the tint (Sandra 0.3 to 1.2 off in a cone, against 6 to 8 with the tint
  alone), and it costs a layer the painted maps have no VRAM for. The
  recommendation is the first, with the second as the later step: see
  "The recommendation".

## 1. Why the night is not drawn

Every `ShowPicture` of a `*Nacht*` picture in the dump, 48 maps, by how it
is placed and what the converter's log says:

    placed by          maps                                    on the console
    a constant         31, 33, 57, 60, 63, 64, 65, 67, 71, 72,   drawn into the map
                       74–79, 81, 84–86, 92–94, 120, 121,        ("drawn into the map: …
                       123–125, 127–129, 131, 135                at 80 %, 'Nacht_…' at 100 %")
    a variable         32, 59, 61, 62, 66, 68, 69, 70            not drawn: "bigger than the
                                                                 plane of objects"

On map 59 the parallel page `EV0136` does, verbatim:

    ControlVars  v14 = event 139's screen X;  v15 = event 139's screen Y
    ControlVars  v14 -= 1;  v15 -= 8
    ShowPicture  1 'Lichteffekt_Eingang_Nacht' at (v14, v15), with the map, transparency 20
    ShowPicture  2 'Nacht_Eingang' at (v14, v15), with the map, transparency 0
    EraseEvent

A picture that goes with the map is placed in screen coordinates at the
moment it is shown, so on a map that scrolls the author anchors it to an
event in the middle of the map, wherever the camera is. `placed()` in
`map_pictures.rs` of the converter (the-advocate) refuses a place "by a
variable", and the pictures fall back to the big ones, which a 369x272
canvas does not fit. Nothing else tints: no page of those maps has a
`TintScreen`, and nothing erases the pictures —RM2k 2000 erases them on a
map change, easyrpg's default `erase_on_map_change`—.

**The same idiom leaves 17 more maps unlit**, in the day: the motel's
rooms (16, 17), the warehouse (22), 48, the dream (58), the mines (102 to
106, 182), the workshop's cellar and vault (136 to 138, 143, 144, 157).
Their masks are lights at an opacity of 35 to 70 %, in the cartridge's
`degradations.json` ("its place is by a variable"). That is not a night
and not this study: draft
[`desert-nightmare-light-masks-placed-by-an-anchor`](https://github.com/lautarodragan/the-advocate-work/blob/main/tasks/desert-nightmare-light-masks-placed-by-an-anchor.md).

**Map 2** has no night pictures: its autostart tints to black, waits a
second, and tints back to 100 in three. EasyRPG captured six seconds after
launch is halfway (a mean of 51 against 125 over the map's part of the
screen); from eight seconds on it is as bright as the console, and its
lamp is off in the original too.

## 2. The patches: a tint for each night

**The fit.** RM2k's night composed in software over the map's panorama
—the mask blended at its opacity as EasyRPG does, the opaque picture with
its transparent colour, each placed where its page puts it—, against the
console's tint over the same panorama —saturation toward the Y'CbCr grey,
then each channel times its value over 100, as `apply_tint` does—, both
at 15 bits, over every pixel but the panorama's holes (where the building
pieces go) and the lit windows. Searched over grey values and
saturation, then each channel apart:

    map  mask (transparency)                 tint               all   dark  cones  windows  day (before)
    59   Lichteffekt_Eingang_Nacht (20)       25, 25, 25,  85    0.47  0.21  4.03   22.9     12.78
    61   Lichteffekt_Werkstatt_Nacht2 (20)    29, 29, 29,  60    1.53  0.52  4.14   19.4     10.03
    61   Lichteffekt_Werkstatt_Nacht (20)     29, 29, 29,  60    1.47  0.51  4.19   19.4     10.09
    62   Lichteffekt_Wohngebiet_Nacht (20)    25, 25, 25,  85    0.21  0.12  4.59   21.8     14.56
    66   Lichteffekt_Polizeirevier_Nacht (20) 25, 25, 25,  85    0.75  0.21  4.80   22.2     11.33
    68   Lichteffekt_Post_Nacht (20)          25, 25, 25,  85    0.78  0.19  4.83   22.3     12.35
    32   Lichteffekt_Motel_Innen2_Nacht (45)  50, 50, 50, 100    0.36  0.19  3.68     —       1.77
    69   Lichteffekt_MotelInnen_Nacht (35)    43, 39, 40, 100    0.42  0.27  4.06     —       2.12
    70   Lichteffekt_Motel_Innen2_Nacht (35)  43, 39, 40, 100    0.45  0.25  4.45     —       2.10

The mean absolute difference per channel, of 31. "Dark" is where the
mask is its plain near-black, "cones" where its light is 30 or more of
255 (1 to 10 % of the map), "windows" the opaque picture's pixels (up to
2 %). Over the dark a tint is as good as the picture —0.1 to 0.5, the
rounding—; the cones and the windows are what it cannot do. The interiors'
"day" is low because their panoramas are mostly the black around the
room.

**The forks.** `desert-nightmare/patches/manual/maps/` in the ports, one
per map, A Blurred Line's rules —a fork is the whole map, the dump's
`.tiles` byte for byte, the `id` inside—; the folder's README says what
each changes. Three kinds of change and nothing else:

* the mask's `ShowPicture` becomes the `TintScreen` of the table, at once
  and without waiting; the variables the page reads stay, and so does map
  68's windmill (`Windrad`, picture 1);
* the `ShowPicture` of `Nacht_*` is gone: a tint has no windows;
* a neutral `TintScreen` goes right before every `Teleport`, since RM2k
  keeps the tint across a map change and the maps behind the doors light
  themselves. Every one of those pages erases the screen before its
  teleport already, so the neutral tint is never seen.

**Measured on the console**, against EasyRPG, each capture aligned inside
the other, the mean per channel at 15 bits of the whole screen:

    against                        59    61    62    66    68    69    70
    the original, before          12.4   8.6  11.6  10.3   9.8   2.5   2.7
    the original, with the forks  0.90  1.36  0.87  1.08  1.37  0.69  0.58
    EasyRPG with the same forks   0.10  0.26  0.10  0.08  0.10  0.07  0.05

**RPG_RT runs the patched maps the same**: the forks built into a copy of
the game with Ditto, EasyRPG Player draws them tinted; through the door
from 59 to 60, map 60 comes out identical with and without them (a
difference of 0.0). The suite has one test, `test_desert_nightmare_nights.py`
in the ports: map 59 entered like a door is tinted and dark. The other
seven were looked at in captures.

## 3. The lights over a tinted night

What a tint leaves out is 1 to 10 % of each map: the cones, where RM2k's
mask is lighter, and the windows, where the opaque picture is. Each way of
putting them back was measured in software at 15 bits against RPG_RT's
night over the whole panorama —not in a ROM: none of them is built—, and
over Sandra's sprite standing in the dark and in a cone's light (its 90th
percentile, 67 to 88 of 255):

    way                                  59 cones  61    62    66    68    69   │ Sandra: dark  cone
    A  the tint alone (the forks)          4.03   4.14  4.59  4.80  4.83  4.06 │        0.48  6.2–7.8
    B  the night drawn into the map        ~0     ~0    ~0    ~0    ~0    ~0   │        5.4   4.0–4.9
    L1 tint + lit cells, untinted          0.24   0.20  0.24  0.12  0.19  0.18 │        0.48  6.2–7.8
       + the character's palette lit       —                                   │        0.48  4.0–4.9
    a  darkness subtracted, 2-bit layer    3.61   5.02  1.26  3.08  3.56  2.46 │        1.5–4.4  4.0–7.7
    b  tint + light added, 2-bit layer     0.50   0.60  0.59  0.59  0.60  0.57 │        0.48  0.3–1.2
    c  tint + HDMA windows, fixed colour   0.74   0.98  0.98  1.03  0.90  1.03 │        0.48  —

The whole map, all pixels: A 0.21 to 1.59, L1 0.11 to 0.39, b 0.13 to
0.61, c 0.14 to 0.71, a 1.42 to 3.33. The pictures of each, for 59 and 61,
are in the captures named at the end.

### B. The night drawn into the map

What the one-screen nights already do: the converter resolves the anchor
—`ControlVars` from an event's screen position plus a constant is the
event's cell, which a parallel page on its own map knows— and draws both
pictures in, as on map 60. The map is RM2k's; **the characters are lit as
by day everywhere** (Sandra 5.4 off in the dark, which is nine tenths of
the map), and it undoes the patches. It is the answer for the 17 day maps
of the draft above, not for the nights.

### L1. Lit cells in palettes the tint skips

The tint is a transformation of CGRAM, `apply_tint`: it walks the map's
background range from `CGRAM_BASE`, the layer above's palettes and the
sprites', and leaves the window's palette and the timer's entries alone.
The 8x8 cells where the tint misses most (a mean over 1 of 31) are 112 to
281 per map, 7 to 17 % of the cells; drawn as RPG_RT's night —the cone or
the window composed in—, in **one or two of the painting's palettes that
`apply_tint` stops short of**, they come out untinted and right: 0.12 to
0.24 over the cones with 30 colours, 0.33 to 0.45 with 15; the windows 0.3
to 1.0 with 30 (61, 4.4 with 30 and 7.5 with 15: its windows are bigger
and yellower).
The 58 to 117 colours those cells want go down to 15 or 30.

* **Cost**: CGRAM, 15 or 30 of the painting's 105 —the painted maps keep
  63 to 81 colours of 130 to 496 today, so the rest of the painting gets
  fewer—. ROM, VRAM and cycles nothing: a painted map already carries a
  tile per cell, and the lit cells are cells. The runtime, one number per
  map: where the tint stops in the background's range.
* **Converter**: the night pictures composed into those cells, at their
  anchor, as `map_pictures` composes them on a one-screen map; and knowing
  which tint the page leaves, to pick the cells. Neither is in it today.
* **What is lost**: the characters. Sandra walking into a cone stays as
  dark as in the street (6 to 8 off). Giving each character a lit copy of
  its palette while it stands on a lit cell (a bit per cell, ~200 bytes a
  map, and a palette swap in the sprites' pass) brings that to 4 to 5 —
  the copy is the untinted palette, brighter than the cone—, and doubles
  the object palettes, which only maps with up to four fit.

### a. The darkness subtracted

A layer drawn by the mask's level, on the subscreen, subtracted from the
main screen —BG3 at 2 bits in mode 1, which the painted maps leave
unused—. Subtraction is not a darkening: a bright wall minus a level
stays bright, a dark sand goes to black. **1.4 to 3.3 over the map, 1.3
to 5 over the cones**, five times the tint's error over the dark; the
earlier measurement on the WC, in [Pictures drawn into the
map](../../../divergences/pictures-drawn-into-the-map.md#measured), found
the same. It reaches the characters with object palettes 4 to 7 and
crushes their dark colours to black (Sandra 1.5 to 4.4 off in the dark, 4
to 8 in a cone). Its costs are the next option's.

### b. The tint, and the light added

The same layer the other way round: the tint darkens everything, sprites
included, as RM2k's picture over them does, and colour math **adds** the
layer's level where the mask is light. Four levels from the mask's light
—0 and three, of 3 to 8 of 31, per map— give **0.50 to 0.60 over the
cones**, with the rings of four steps showing in the gradient; and it is
the only way that **lights a character** walking into a cone: Sandra 0.3
to 1.2 off there, against 6 to 8 with the tint alone.

* **VRAM**: the layer's distinct 8x8 cells, 70 to 222 2-bit tiles (1.1 to
  3.5 KB), and a 32x32 tilemap (2 KB) filled as a ring like the painting's.
  The painted layout (`VENTANA_PINTADA`) leaves 1,024 words free at $5C00
  and ~500 past the painting's slots, at $4DF0: the tilemap, and 66 tiles.
  The rest would come out of the layer above's 128 tiles, which carry the
  roofs' edges in front of the hero: half of them for 59 and 68, and 61
  does not fit even then.
* **CGRAM**: BG3's palettes in mode 1 are entries 0 to 31, which are the
  window's palette 0 and the painting's palette 1 —where a face's
  colours also go, by HDMA, on the window's lines—. One palette of four
  entries for the levels, out of the painting's, and `apply_tint` has to
  skip it.
* **Cycles**: the colour math is the PPU's; the tilemap's ring is one
  more column or row a vblank when the camera crosses a cell, 64 bytes,
  where a painted row and column already end the vblank at line 251 of
  262.
* **The characters**: only objects with palettes 4 to 7 take part in
  colour math. The night maps' characters take 1 to 6 palettes from the
  bottom (62, 66, 68 and 70 four or fewer; 32, 59, 61 and 69 five or six),
  so on four of the eight some characters would stay unlit in a cone; the
  pictures in sprites and the bush take theirs from 4 to 7 too.
* **The window**: the layer has to go off on the window's lines, as the
  window's own HDMA does with BG1 and BG2.

### c. HDMA windows and a fixed colour

The two window masks shaped line by line by HDMA, and the fixed colour
added inside them: no layer, no VRAM, no CGRAM. The edges are crisp and
the gradient goes: **0.74 to 1.03 over the cones**. Two windows a line are
enough on 59, 62, 66 and 68; on 61 38 of 304 lines cross three cones or
more, on 69 14 of 240, and the third is dropped. It takes an HDMA channel,
and every one has a use —the general DMA 0 and 7, the message window 1
and 2, its swaps 3 and 6, the big pictures 4, a face 5—, so it would share
one of those that run only at times; and the window registers, which the
message window's masks use. Objects
with palettes 4 to 7 are lit inside, with a flat colour.

### Sprites

A cone as objects is 64x64 pixels, 64 tiles; a painted map has 256 sprite
tiles and 59 and 61 use 252. Not measured further.

## The recommendation

**L1 first**: the cones and the windows drawn into the cells the tint
misses, in palettes it skips. It is the closest to RPG_RT over the map
(0.1 to 0.5), costs only colours, and needs the converter to resolve the
anchor, which is also what the 17 day maps of the draft need. What it
leaves out is the characters in a cone, which stay dark —as every
interior's already do, see [Pictures drawn into the
map](../../../divergences/pictures-drawn-into-the-map.md)—.

**b next, if the characters matter**: it is the only one that lights
them, but it needs VRAM the painted layout does not have and object
palettes four of the eight maps do not leave.

Neither is small: both are a converter feature, and L1 a runtime number
too. Not implemented: task
[`desert-nightmare-lit-cells-over-a-tinted-night`](https://github.com/lautarodragan/the-advocate-work/blob/main/tasks/desert-nightmare-lit-cells-over-a-tinted-night.md).

## How it was measured

* **The sweep**: every `ShowPicture` whose picture names `Nacht` in the
  dump's `data/maps/`, by its position parameter, against the lines of
  `converter.log` and `degradations.json` of the `release` build.
* **The captures**: the console's, 60 ticks after the harness's door jump
  —`go_to_map`, which clears the tint, so what is seen is the page's—;
  EasyRPG Player's, a new game started on the map at the same cell
  (`--start-map-id`, `--start-position`), 8 seconds after launch, halved
  to 320x240. Aligned by the shift that matches best, and compared over
  the overlap.
* **The fit and the options**: RPG_RT's night composed over the panorama
  file in software, as easyrpg blends a picture (opacity times 255 over
  100, the transparent colour skipped), placed at the anchor event's
  screen position less the page's subtractions; the console's tint as
  `apply_tint` computes it. The options' layers quantised to four levels
  by the mask's light (30 and its 50th and 85th percentiles over the
  cones), each level the median that fits best; the lit cells' colours by
  median cut. Sandra's sprite is character 1 of `Sandra`, facing down.
  None of this ran on a console, and the panorama has no characters,
  roofs or pieces in it.
* **The forks in RPG_RT**: the same script edited the game's `.lmu` with
  `ditto read` and `ditto write`, and `ditto unpack` of that copy gives
  the same event commands as the forks.
