# What the debug screen does not model

## What

The pause menu's debug screen —see [The pause menu and its
options](../internals/the-pause-menu.md)— is RPG_RT 2000's F9 screen, and
differs from it in:

* **How it is reached**: the pause menu's last option, Debug, and not F9,
  which the console does not have. RPG_RT opens it from the map only in a
  test play; here it is in any cartridge built with `[debug]
  debug_screen`.
* **A variable is changed by steps**: left and right by one, L and R by ten.
  RPG_RT opens a number input over the list, a sign and seven digits.
* **A variable is sixteen bits**, as everywhere in the port —see [Variables
  saturate at sixteen bits](variables-saturate-at-16-bits.md)—: a step past
  32767 or -32768 stays there.
* **A variable the cartridge does not keep shows a grey `-`** and does not
  change: the cartridge gives a slot only to the variables a page of its
  maps names, and nothing would read the others. RPG_RT shows every one.
  The same with a switch past the cartridge's table —the highest the game
  names—, which shows a grey `[OFF]`.
* **It is 512 pixels wide**, the console's high-resolution mode, so that
  the names fit: the letters are RPG_RT's pixels at half the width, and
  the windows keep their look but, in that mode, only three colours a
  cell, which a System with more pays. A name is cut only past what the
  list's window holds, twice as many letters as at 256 wide —none of
  A Blurred Line's is—, and a letter the screens' font does not have goes
  as its plain stand-in or a `?`.
* **The list takes about half a second** to show after a step that
  changes it whole: the screens' text layer composes it a few rows per
  frame, and its tiles do not hold two pages, so the rows composed so far
  show and the others keep what they said until their turn —never blank:
  until 2026-10-20 they went blank, ranges included; see
  [the task](https://github.com/lautarodragan/the-advocate-work/blob/main/tasks/the-debug-screen-redraws-and-goes-wide.md)—.
  The ranges beside the list are not composed again. RPG_RT redraws it
  at once.
* **Its sounds** —RPG_RT's cursor, decision and cancel— are not played.

## Why

It is a tool for whoever ports a game, not part of the game: what it has to
do is change the state, and do it the way RPG_RT does —the change takes
effect when the map comes back—. A number input is a second screen for what
a few presses do.
