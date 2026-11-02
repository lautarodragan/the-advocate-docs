# A loop that never waits nor ends does not compile

## What

`Loop`, `BreakLoop` and `EndLoop` compile since 2026-10-12, as jumps:
without the Maniac patch RM2k keeps no state for a loop —easyrpg's
`CommandLoop`, `CommandEndLoop` and `CommandBreakLoop`—, so `EndLoop` goes
back to after the first `Loop` at its own indentation, and `BreakLoop` to
after the first `EndLoop` that follows it, whatever its indentation, which
is RPG_RT's bug and easyrpg copies it.

**A `Loop` whose body has nothing that lets a frame go by and no way out
does not compile**, and the page it is in is left out like any page with a
command the ROM does not do. What lets a frame go by is a `Wait`, a
message, a choice, a number to input, a `KeyInputProc` that waits for a
key or a `MovePicture` that waits for its move. A way out is a `BreakLoop`
that lands past the loop's own `EndLoop` —one inside an inner loop ends
only that one— or a `JumpToLabel` to a label outside it: since 2026-11-01
a loop with one of those compiles without waiting, and runs all its turns
in one frame, as RPG_RT does with fewer than ten thousand commands.

**The wait counts even when what comes with it is not done.** A
`MovePicture` the console cannot do —a picture the map does not carry, a
move longer than its byte, a big picture slid sideways— is accepted
without being done, and since 2026-10-20 its wait still compiles, as in
easyrpg's `CommandMovePicture`, which waits whatever happened to the
picture. Until then the wait went with the move, and a loop counted as
waiting was left with nothing that ends a frame: Vampires Dawn's Tradan
hung on its first parallel page —see
[the task](https://github.com/lautarodragan/the-advocate-work/blob/main/tasks/vampires-dawn-tradan-hangs-on-arriving-from-the-start.md)—.

## Why

RPG_RT runs at most ten thousand commands of an interpreter per frame —it
is `loop_limit` in easyrpg's `Game_Interpreter::Update`— and then lets the
frame go by, so a loop that never waits slows the game down
but does not stop it. The ROM's interpreter has no such cap: the same loop
would never return, and the console would hang.

## How it looks today

Vampires Dawn's loops almost all wait: the rain, day and night, the mines,
the clock, its menus. Zounds' do not: its mining pay (map 134), the
smelting oven (map 133), the mining merchant's scene (map 57) and its
next-level experience (common events 51, 52 and 54) count items or
variables down and break, and they were left out until 2026-11-01. A loop
that waits only on some of its branches compiles, and hangs if it takes
the others forever; so does one that breaks only on a condition that is
never met, where RPG_RT would go on drawing.

## When it stops being enough

A game with a loop that neither waits nor can break —one that ends only
when a parallel page or the player changes something— would need a cap on
the interpreter's commands per frame, like RPG_RT's.

## Measured

`a_loop_that_never_waits_does_not_compile`,
`a_loop_that_can_break_compiles_without_waiting`, with the two others on
loops, and `a_move_not_done_still_waits`, in
`rm2k-snes/events/src/compile.rs` in the-advocate. The ROM tests that run
a loop are `tests/test_vampires_dawn_tradan.py` in the-advocate-ports,
Tradan's parallel loop, whose move is not done, ticking for 900 frames;
and `test_the_mining_is_paid` in `tests/test_zounds.py`, four loops that
break without waiting.
