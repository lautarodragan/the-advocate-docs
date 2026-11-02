# Too many characters on one line

## What

Every character on the map —an event drawn from a CharSet, the hero— is
one 32x32 object, and the console draws at most 32 objects and 34
slivers of eight pixels on each line. A 32x32 costs four slivers on each
of its 32 lines, drawn or not, so **eight and a half characters side by
side fill a line**. Past that the console drops slivers, and the ones it
drops are the frontmost objects' —the lowest in the OAM—: the OAM goes
from front to back, so what goes missing is what is in front, which on a
row of equal events is the row further down and, beside them, the hero.
RM2k draws them all.

It was seen where a map lines many events up: Zounds' map 96, *Arriba
del Cuartito*, has two rows of seventeen floor switches one above the
other, and on screen sixteen of each. The first row showed eight switches
and the ninth cut in half, the row below none on the lines it shares with
the first, and a character standing beside them lost what went over.
Since 2026-11-02 Zounds draws them into the background —see [Still events
drawn into the background](still-events-drawn-into-the-background.md)—,
and on map 96 no line has more than 16 slivers.

## Measured

On Zounds' map 96 in snes9x, which drops slivers as the console does,
counting the objects of the OAM the ROM builds, on 2026-11-02 with
the-advocate at `0ba09da4` plus the branch `cardon-map-96`: with the
camera on the switch column at x 8, up to 84 slivers on one line and 48
lines over 34; with it on the two rows, up to 128 slivers, 34 objects on
16 lines and 48 lines over. With the switches in the background, the same
day, at most 16.

## Why

The console's sprites are what they are. Drawing a 16x16 event as four
8x8 objects would halve its slivers, but it takes four of the 128 objects
the whole screen has, and that screen of map 96 shows more than forty
switches. What fixes it is **not drawing them as objects**: a game with
`[graphics] posed_events` paints the still ones into the background and
writes their cell again when a page or a route changes their pose —see
[Still events drawn into the background](still-events-drawn-into-the-background.md)—.
What it leaves as objects —one that walks, animates, goes above the hero
or draws past its cell— still counts here.

## How to reproduce it

Line up more than eight characters that walk, or that the background
does not take, on one row of a map. `tests/test_zounds_map_96_switches.py`,
in the ports, looks at map 96's lines in the OAM.
