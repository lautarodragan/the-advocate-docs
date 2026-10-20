# The test page switches itself off

## What

**The test page switches itself off**, by turning on a switch in its first
command —1023, unless the game names it—, with a second, empty page that
requires it. Its event is the map's first free index, not a high number.

## Why

It is the idiom with which the game's pages deactivate themselves, and
nothing exercised it. Without it, now that autorun pages repeat, the test
page would restart its own route forever.

As for the index: the ROM's sweeps go from zero to `ev_total`, so a high
number would be left out.

## How it applies

Every cartridge with cases carries the test pages, the release too with
its empty case, so the switch is on from the first frame of every game.
**It has to be one the game does not name.** 1023 was measured against A
Blurred Line, which names up to 1008, and it was the switch for every game
until 2026-10-19, when The Frozen World's first battle showed the wizard
instead of the plant that started it: its battle's first monster shows the
wizard on a page that asks for 1023, which RM2k leaves off until a late
fight. See
[`frozen-world-the-first-battle-shows-the-wrong-enemy`](https://github.com/lautarodragan/the-advocate-work/blob/main/tasks/frozen-world-the-first-battle-shows-the-wrong-enemy.md).

The converter asks the event compiler's scan for 1023, and the scan answers
it or the first switch above it that nothing names —a page condition, a
command, a range whole, a common event's trigger, a troop page, the
database, the test pages' own commands—. The cartridge says which as the
symbol `TEST_SWITCH`, which a harness reads instead of assuming 1023.
