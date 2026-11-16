# Is there a better palette grouping?

*2026-11-16. Measured on the-advocate `705d24f5` and the-advocate-ports
`299088c`, A Blurred Line's `release`. It is a snapshot: it is not
updated.*

On map 3, Emitter Room, the `Future1` tile Lysander used as Core's blood on
the floor, at (46, 19), comes out brown on the console. The question was
whether Adeleine could group the colours better in general, or whether A
Blurred Line keeps targeted fixes —the blood's own is the task
[`abl-core-s-blood-turns-brown`](https://github.com/lautarodragan/the-advocate-work/blob/main/tasks/abl-core-s-blood-turns-brown.md)—.
The study is the task
[`abl-is-there-a-better-palette-grouping`](https://github.com/lautarodragan/the-advocate-work/blob/main/tasks/abl-is-there-a-better-palette-grouping.md).

**The answer: yes, there is an obvious one, and it is not a new
objective.** Adeleine's grouping stops at the first local minimum its
start leads to, and on map 3 that start spends a whole palette on white.
Trying 21 starts and keeping the one its own cost likes best gives the
blood back its red and lowers the error of every map that groups, with
the same objective, the same distance and nothing invented.

## Where grouping matters in A Blurred Line

In mode 3 the map is 8 bits per pixel: its colours are one pool and are
never grouped (see [The converter decides the CGRAM
split](../../../internals/cgram.md)). Grouping into palettes of fifteen
—`reduce_colors` in `snes/adeleine/src/pack.rs`— happens on:

* **the five maps that go in mode 1**, whose map is 4 bits per pixel in
  five palettes: 3, 6, 32, 105 and 255 (`rm2k-snes/wally/src/background/mode1.rs`);
* **the layer above the hero** of every map, which merges on 26 maps, 15
  of them with a single palette, where no grouping can help;
* the pictures, the screens and the battle, which were not measured here.

The characters are not grouped by Adeleine: wally splits them by its own
rule, and one that fits in no palette is lent one —see [The palette a
character is lent](../../../research/2026-10-17-the-palette-a-character-is-lent.md)—.

## Map 3's blood

Map 3 has 75 colours for 5 palettes of 15: 75 entries. Adeleine keeps 36
distinct colours. Its five palettes, as the release has them:

| palette | colours | what |
|---|---|---|
| 0 | 15 | the dark floor, its greys and teals, and two browns |
| 1 | **1** | white |
| 2 | 12 | the green acid and some greys |
| 3 | 15 | greys and teals |
| 4 | 15 | greys, teals, greens |

The blood's four 8x8 cells land in palette 0: what they share with it is
the floor's near-blacks and a teal, and Adeleine's first grouping places a
cell by the mean of its colours, which here is the floor's. Palette 0
chooses its fifteen colours by pixels over every cell of its group, and
the blood's reds are about 120 pixels of the map's 1.44 million, all in
that one tile: none is chosen. Each red then goes to its nearest in the
palette by the luma distance —squared channel differences weighted
30/59/11—, which counts red at 30 and green at 59:

| blood colour (15 bits) | pixels | becomes | dE2000 |
|---|---|---|---|
| `[25,10,10]` | 37 | `[20,15,10]` | 22.3 |
| `[30,12,12]` | 15 | `[20,15,10]` | 24.4 |
| `[30,16,13]` | 15 | `[20,15,10]` | 20.2 |
| `[17,6,4]` | 29 | `[15,10,7]` | 12.5 |
| `[19,8,6]` | 8 | `[15,10,7]` | 11.5 |

**The browns are the rust specks of the teal wall panels** —from (33, 42)
down, and the riveted edges of the room—: `[15,10,7]` covers 708 pixels
and `[20,15,10]` 253, so palette 0 keeps them, and they are the only
warm colours it has. The tile's mean colour difference is 6.4 to 14.7
dE2000 per cell.

## The variants, over every grouping call of the release

Each call wally makes to `reduce_colors` was written out with the pixel
histogram of every place, and replayed with variants of Adeleine. The
error is the mean CIEDE2000 per pixel against the picture at 15 bits,
summed over the calls that merge (31; a call that fits loses nothing in any
variant). Lower is better.

| variant | all calls | mode-1 maps | layer above | calls better / worse |
|---|---|---|---|---|
| today | 1,150,298 | 904,760 | 245,538 | |
| distance in Lab | −20% | −19% | −22% | 26 / 4 |
| distance in CIEDE2000 | −24% | −25% | −22% | 25 / 5 |
| 50 refinement rounds instead of 12 | 0% | 0% | 0% | 0 / 0 |
| **21 starts, best by its own cost** | **−49%** | **−62%** | 0% | **9 / 0** |
| 21 starts, distance in Lab | −53% | −61% | −23% | 28 / 2 |
| pixels weighted by their square root | +52% | +55% | +44% | 2 / 25 |
| pixels weighted up by chroma (1 + C/20) | −11% | −16% | +7% | 8 / 16 |
| Lab and chroma | −23.5% | −24% | −21% | 16 / 14 |
| one more palette | −34% | −32% | −42% | 29 / 2 |
| 21 starts and one more palette | −62% | −60% | −71% | 31 / 0 |
| colours moved to their group's mean | +1% | +1% | +1% | 6 / 18 |
| 21 starts, colours moved | −47% | −60% | +1% | 11 / 13 |

What it says:

* **The search is what is short, not the objective.** More rounds change
  nothing: the grouping has already settled. Other starts —random cells
  as the first centres, 20 of them, plus today's— find groupings the
  same cost function prefers, and none of the nine multi-palette calls
  they touch gets worse. Map 3 keeps 40 colours instead of 36.
* **Weighting rare colours up does not pay**: by square root everything
  gets worse; by chroma, the reds win and the floor loses.
* **The distance seeing hue helps less than the starts, and costs on the
  layer above of four maps** (269 to 273, the same room).
* **Inventing colours does not pay** as tried: each kept colour moved to
  the pixel-weighted mean, in RGB at 15 bits, of what it stands for in its
  palette. The colours that were exact move too, and the means of a grey
  and a red are neither. Measured cheaply, one step and with no
  threshold; a more careful version was not tried.
* **More palettes** for the map would be a layout change: in mode 1 the
  map already takes five of eight, and the layer above one.

Which distance each step uses today: `try_pack`, which only asks whether
the tiles fit, none; the first grouping, the plain RGB distance between
the cells' mean colours; choosing each palette's colours, reassigning the
cells and each colour's nearest, the luma distance; the 8-bit background's
merges (`merge_least_visible`), the luma distance; a lent character's
nearest, CIE Lab since 2026-10-17. With the distance alone in Lab or
CIEDE2000, map 3's blood still lands in palette 0, and its reds go to
`[19,8,6]` or to the browns: a darker red-brown instead of a light
brown, 4 to 14 dE2000 a cell. Oklab, which lost to Lab on the lending, was
not tried here.

On the pictures wally writes —the release's own pictures, with the
variant built into Adeleine—, the mean dE2000 per pixel of each mode-1
map, against its picture at 15 bits:

| map | today | 21 starts | 16x16 cells worse by more than 2 |
|---|---|---|---|
| 3, Emitter Room | 0.090 | 0.059 | 0 of 5,625 |
| 6, CoreRetro II | 0.001 | 0.000 | 0 |
| 32, Sector 27 Warehouse | 0.099 | 0.023 | 2 of 2,500 (+2.1) |
| 105, Mountain Path | 0.057 | 0.008 | 0 |
| 255, Under the Bridge | 0.015 | 0.005 | 0 |

With 21 starts the blood lands in a palette with `[30,12,12]` and
`[12,6,6]`: its cells are 3.1 to 5.5 dE2000 instead of 6.4 to 14.7, red on
screen.

**What it costs**: only the calls that merge run it again —a call that
fits stops after the first—, about four seconds for map 3's grouping
instead of a quarter, and wally makes it three times per map. The answer
stays deterministic: the starts come from a fixed sequence, and ties are
broken as they are today.

## Map 8's signs: another mechanism

The signs taro saw on Lashe City —events drawn with the ChipSet's tile 53,
a monitor with a green screen— are characters, not the background. On map
8 (EV0006, EV0007, EV0018, EV0027, EV0031) the tile is **lent** a palette:
28 characters for 8 palettes. The palette that costs least among those it
can borrow is an orange and brown one, and the screen's two greens
`[5,27,18]` and `[5,24,17]`, 20 pixels, go to a grey `[18,18,18]` and a
dark teal: dE2000 30 and 36, and 6.5 over the sign. That is the lending of
[The palette a character is
lent](../../../research/2026-10-17-the-palette-a-character-is-lent.md),
whose step 3 —grouping the characters that do not fit and quantising them
together— was measured there and not built. On map 9 (EV0024 and EV0026,
tiles 52 and 53) both signs get palettes of their own and the converter
draws them with their exact colours: what looks off there is not the
palette, and would need a capture.

## How it was measured

A throwaway wally wrote each `reduce_colors` call's places, with the pixels
of each colour, and the layers before any colour is lost. ABL's release
was built through a shim that kept each map's request, and wally run again
on it, with today's Adeleine —its pictures equal to the release's— and
with the variant. A harness replayed the calls with a copy of `pack.rs`
whose distance, weights, rounds, starts and limit could be changed. The
captures —original, today, 21 starts— are where taro looks at them.
