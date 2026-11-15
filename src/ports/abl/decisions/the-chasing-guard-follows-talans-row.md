# The chasing guard follows Talan's row

## What

**On Mountain Path (map 105) the port does not run the original's events:
it fixes a bug of the game.** When Talan steps on the trigger, an Agency
guard (event 5) chases him, and a parallel event (event 6) picks the
guard's page by Talan's row: it copies the row into variable 53, and turns
switch 328 on if the row is 18 or more —the planks below the road— and 329
if not. Page 2 of the guard asks for 328 and walks a route that zigzags
down first; page 3 asks for 329 and zigzags up first.

**The original never turns either switch off.** Talan is on row 17 when
the chase starts, so 329 is on from its first frame, and page 3, the
higher one, wins for the whole chase, also once Talan goes down to the
planks and 328 turns on too. In the port, event 6 also turns 329 off where
it turns 328 on, and 328 off where it turns 329 on: the guard's page
follows the row Talan is on.

It is the first change of this kind: the other forks of ABL's maps change
what a map costs the cartridge, not what the game does.

## Why

taro asked for it on 2026-11-15, after reading the event in RPG Maker:

> Could you apply a patch that fixes this bug directly in A Blurred Line?
> I suppose it could be fixed just by adding two commands: one that turns
> the switch off, for each of the two switches, in the two branches of
> the `if`, right?

It was measured first that the bug is the game's and not the port's. In
RPG_RT under Wine, with an event that reads the cells and the switches
every frame and Talan walking the upper path, the lower one, or standing
still, the guard steps on the same cells as in the ROM, frame for frame,
and catches Talan in the same places: 329 is on all the chase, 328 turns
on when Talan reaches row 18, and the guard's route is page 3's all the
way. That is in
[`abl-the-chase-on-the-lower-path`](https://github.com/lautarodragan/the-advocate-work/blob/main/tasks/abl-the-chase-on-the-lower-path.md).

The event reads as meant to do what the patch makes it do: two switches
picked by a row, two pages with mirrored routes, and page 2 reached
otherwise only if Talan were on the planks when the chase starts, which
the trigger, on row 17, does not allow. What changes playing is little: the guard still
catches Talan on every path walked, and on most of them in the same
place; where Talan goes down to the planks and the guard is behind him,
the guard now heads down after him first.

## How it applies

The fork is `abl/patches/manual/maps/Map0105.json` and its `.tiles`, in
the-advocate-ports: the dump's map with the two `ControlSwitches` added and
nothing else changed, which `diff` against `abl/data/maps/` shows; its
`README.md` lists it with the others.

The patched game was measured too: the same two commands applied to the
original's map 105 and run in RPG_RT, against the ROM, over six walks —one
of them down to row 18 and back up, so that the page goes 3, 2 and 3—.
The guard's cell and both switches are the same on every frame, and the
battle starts on the same frame. That includes what a page change does to
the guard's own route: it starts over from its first command, in both.
`test_the_guard_of_105_follows_the_row_talan_is_on`, in
`abl/tests/abl/test_paradise.py`, checks two of those walks against
RPG_RT's numbers, and says that it diverges from the original on purpose.
The task is
[`abl-the-chase-follows-the-row-talan-is-on`](https://github.com/lautarodragan/the-advocate-work/blob/main/tasks/abl-the-chase-follows-the-row-talan-is-on.md).

If someone compares map 105 with the original and finds event 6
different: this is why, and undoing it brings the bug back.
