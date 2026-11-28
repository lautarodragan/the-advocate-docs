# How it is tested

Nothing is considered right because it looks right. A test builds a
cartridge, runs it in an emulator, and **compares the screen pixel by
pixel at 15 bits**, which is the console's real color depth, against a
frozen capture (a *golden*).

Three rules come with that:

- **Changing a golden is deliberate.** It goes into version control, it is
  looked at, and the commit says why the picture changed.
- **A golden catches regressions, not birth defects.** A feature that is
  wrong from day one bakes its bug into its golden. What balances that is
  someone actually playing the game.
- **Two emulators, compared to each other.** One of them starts with
  garbage in work RAM and the other with zeros, so a difference between
  them means the ROM reads memory it never wrote.

## The test game

Most of those tests live in A Blurred Line's suite and test what RPG Maker
2000 does, using that game as a stage. They are moving to a game of our
own, in the-advocate, built from text with free assets and run with `just
test-game`; where that stands is [its progress
page](progress/the-test-game.md).

## The other games

A Blurred Line's suite runs only A Blurred Line, so a change to the tools
could break another port without anyone noticing. The root of
the-advocate-ports has a `justfile` of its own for that:

    just boot            # every port but A Blurred Line
    just boot -k sunset  # one of them

For each game it builds the cartridge from its `advocate.toml` —only if
it is stale— and plays it in snes9x and in bsnes until the player has
control: it presses A every few ticks while an event page runs, which goes
through the title screen, the intro, the messages and any question by its
first option. It fails if the game does not build, if the player never gets
control, if that happens on another map than the one recorded in the test,
if the hero cannot walk, or if the two emulators disagree. Beyond that it
does not look at how the maps are drawn: goldens of seven games would bake
their birth defects in and multiply what has to be reviewed.

The exception is the painted maps of Sunset Over Imdahl, the only game with
any: they are the only path of the engine A Blurred Line never runs, so a
square and a bar have goldens, still and walking with the camera moving.

It is not part of A Blurred Line's suite —one game alone takes a couple of
minutes to build from scratch—: it is run before handing a build over, or
after a large change to the tools.

## The dump's round trip

What makes a game's `data/` the game, and not an approximate reading of
it, is that Ditto writes back from it the very bytes it read: the exact
dump packed again gives every `.lmu`, `.ldb` and `.lmt` equal to the one
it came from. No test asks it: `advocate import` checks its own, every
time, and the recipes that import run it from the two sides —taro's,
since 2026-11-25: no test runs a binary of ours—.

- **The test game's** `just game`, which `just test-game` runs first: its
  RM2k files are written by `advocate export` from a dump written by
  hand, and imported back, so it proves Ditto reads what Ditto writes. It runs in CI, where no one
  else's game can go.
- **Each port's** `just dump`: over the game's original, it proves Ditto
  reads what RPG Maker wrote —2003 too—, every time `data/` is written.

Until that day both were a `test_round_trip.py`, one in `just test-game`
and one in the ports' `just boot`, once per game; until 2026-11-27 the
recipes ran `ditto unpack` and `ditto pack` and compared the files
themselves.

Ditto's own tests ask the same of the library, over whatever game
`DITTO_CORPUS` names: see its README.
