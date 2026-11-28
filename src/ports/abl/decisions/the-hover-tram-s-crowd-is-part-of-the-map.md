# The Hover-Tram's crowd is part of the map

## What

**On the Hover-Tram (map 13) the port draws the passengers who never move
as tiles of the map, not as characters.** The map has 27 people, in two
rows: thirteen seated (y=5) and fourteen standing (y=9). Twenty-one of
them never move —no command of the
game names them— and in the port they are drawn into upper tiles of a
ChipSet of the map's own, a copy of `BLChip1` with the poses added; their
events are gone. The six that the scene moves stay characters: `Jump1`
and `Jump2`, who jump in their seats, the three `Leave` who get off,
and the thief, `Stash`. `Leave1` never leaves: its name says the author
meant it to, but no command names it, and it is baked with the others.

Each person is drawn as RM2k draws it: the seated ones' heads over the
seat backs, which RM2k draws below characters; the one at (3,5), whose
pose is three pixels wider than its cell on each side, with those pixels
on the empty cell to its left and behind its neighbour on the right; and
every one on the middle pattern, which is the one RM2k shows for a
character that does not have a fixed graphic, whatever the page says
(the ROM does not, elsewhere: see the port's known issues).

## Why

taro, playing: not every character on the Hover-Tram was drawn, because
there were too many sprites. The console draws 34 slivers of 8 pixels on
a line, a character takes four, and the standing row was fourteen
characters, 56 slivers. Baking still events into the background in
general ([Still events drawn into the background](../../../divergences/still-events-drawn-into-the-background.md))
took one seated person of this map: the standing ones' heads fall on the
aisle, where people walk behind them, and the seated ones were held back
by the game's recalls of the hero's place and by the colours. A patch of
the map is what taro had planned from the start.

Since 2026-11-28 the recalls no longer hold the seated ones back —`reach`
knows the trip runs once, from where the hero first arrives—, but the
converter alone still does not reach what the patch draws: built without
it, two of the seven still seated people go into the background, the
other five stay objects for the panorama's colours, and so do the
thirteen standing ones, whose heads are over the aisle; the lines go over
again. See
[`reach-does-not-know-a-scene-runs-once`](https://github.com/lautarodragan/the-advocate-work/blob/main/tasks/reach-does-not-know-a-scene-runs-once.md).

With it, the seated row is six characters on a line and the standing row
one, and the runtime's count of frames the PPU dropped objects is 0 over
the whole trip.

## What it costs

* **The panorama's colours.** The people's colours (56 more) go to the
  background, which shares its palettes with the evening outside the
  windows: the panorama keeps 15 colours where it kept 46. It is seen
  through three small windows, and measured against the picture it is
  0.83 levels of 15-bit colour off per pixel on average, where it was
  0.77. The people keep every colour.
* **Who is in front where they touch.** A baked person is under every
  character. Where RM2k draws a person in front of a character beside it
  —`Leave2`, standing at (8,9), draws two pixels of its side over
  `EV0022`, which RM2k draws over them—, the port draws it the other way
  round: seven pixels, until `Leave2` leaves. The same with the thief
  walking over the seated row: RM2k draws the person to his right over
  his four side pixels.
* **Nobody walks there freely.** The trip's scene holds the hero from his
  arrival until he leaves by the door, and no one walks on the row behind
  the standing people, where a baked head would go under a walker instead
  of over. If the map is ever given back to the player, that is the first
  thing to look at.

## How it applies

The map's fork and its ChipSet, 57, are `abl/patches/manual/maps/Map0013.*`
and `abl/patches/manual/ChipSet/0057.json` in the ports, with the recipe's
`cells` (`from_charset` composing over a background) described in its
`patches/manual/README.md`. RPG_RT running the same patch under Wine shows
every stop of the scene as the original does, but for `Leave2`'s two
pixels. The task:
[`abl-the-hover-tram-s-crowd-is-cut`](https://github.com/lautarodragan/the-advocate-work/blob/main/tasks/abl-the-hover-tram-s-crowd-is-cut.md).
