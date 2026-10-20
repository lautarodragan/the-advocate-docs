# What message escapes do not model

## What

A message's text carries RM2k's escapes, a backslash and a letter. The
console does some of them as RM2k does and **drops the rest**: the box
compiles without them, shows its text, and each one dropped is listed in
the build's `degradations.json` as a `message-escape-dropped` loss of its
command, with the escape as it was written.

Done as in RM2k:

* `\|` and `\^` at the end of the box —its tail—: a second's pause once the
  box is typed, and closing on its own. A `\^` anywhere else closes the box
  on its own too, at its end, which is where RPG_RT closes it wherever it
  is written.
* `\.`, `\|` and `\!` in the middle: the typing stops a quarter of a
  second —sixteen frames, and one more per speed past 16—, a second, or
  until a key.
* `\s[n]`, the typing's speed from there on, and `\>` and `\<`, the instant
  text: see [The pace](#the-pace), below.
* `\c[n]`, the text's colour from there on, one of the System's twenty: see
  [The colour](#the-colour), below.
* `\\`, a literal backslash; `\V[n]`, a variable's value; `\N[n]`, an
  actor's name —see
  [What names in messages do not model](what-names-in-messages-do-not-model.md)—.

Dropped, and what is lost with each one:

* `\$`, the gold window, which does not come up.
* `\_`, the half space: the letters on both sides of it come together.
* A `\V[n]` or `\N[n]` the converter cannot write —a variable inside a
  variable, an actor past 255—: the value or the name. And a `\c[n]` or an
  `\s[n]` whose argument carries a `\V[n]`, which RPG_RT reads when the
  box is typed: the colour or the speed.
* The colour, the speed and the instant text in a box that is not a
  `ShowMessage`'s —a choice's options, an `InputNumber`'s, an inn's, a
  troop page's—, which the console draws plain.
* Any other letter, and a backslash at the end of a line: RPG_RT draws
  nothing for them either, and they are listed so that a patch's code or a
  slip in the game is seen.

An escape that takes an argument reads it as RPG_RT does: up to the next
`]`, past a `\V[n]` inside it, or to the end of the line if there is none;
the value is its digits up to the first thing that is not one, and without
a `[` it is 0. A colour past 19 is 0 and a speed is taken to 1 to 20, as in
RPG_RT. A letter RPG_RT does not know takes no argument, so in `\q[3]` the
`[3]` is drawn, as in RPG_RT.

When the converter reflows a box for the console's window, what the box
drops goes at the start of its first box, where the compiler finds it to
list it: its place among the words does not survive the reflow. The
colours, the speeds and the waits keep theirs, and a box the reflow cut
starts in the colour and at the speed the one before it ended in.

### The pace

RM2k types a box letter by letter in a window already up, and the console
does too, at the same pace: at speed 1, the one a page starts at, two
letters a frame; at speed `n`, one every `n / 2 + 1` frames —easyrpg's
`SetWaitForCharacter`, which copies RPG_RT's—. What is not the same:

* **The instant text goes at five letters a frame**, the composer's most,
  where RPG_RT types the whole stretch in one.
* **The window opens after four frames, without an animation**: the box's
  four lines are blanked first, a frame each, and RPG_RT's window opens
  over some eight frames with its contents already clear.
* **A box is done on the frame its last line is typed**: RPG_RT waits two
  more frames after the last letter before the page takes its tail.
* **A box drawn again with its window up is not typed**: a choice's cursor
  moving redraws the box a line at a time, as before.

### The colour

**A box's colours take one palette entry, its ink.** RM2k draws a colour
with the System's 16x16 block, a gradient of some eight shades; the
console's window is palette 0, the maps' backgrounds start right after it,
and Systems that fill its fifteen entries are common. So:

* **The first colour other than 0 a box types takes the ink**, drawn with
  one shade: the mean of the block's rows, weighted by how many of the
  font's pixels each row has. A row of the colour that one of the window's
  own entries is nearer to takes that entry, so some of the gradient
  survives where the window has the shades.
* **Another colour in the same box is drawn with the window's nearest
  entries**, row by row, the way an option nobody can pick is greyed: a
  gold among the test game's blues comes out nearly white. It is rare: of
  Zounds' 7,142 lines with letters, 39 carry two colours other than 0, and
  10 of Vampires Dawn's 49,488.
* **The window gives up one of its own colours for the ink**, in a game
  whose boxes use a colour: Adeleine merges the pair that costs least, and
  the build writes it in the game's patches like any colour a System
  loses. In the test game it is the darkest band of the window's gradient,
  on its bottom eight lines. The maps lose nothing: the entries the window
  takes are the same with the ink as without it.

## Why

Until 2026-10-15 any escape but the tail, the waits, `\c[n]` and `\s[n]`
left the whole page out, which kept a game from being played until each
escape was done: Vampires Dawn writes `\_` in hundreds of boxes. Drawing an
escape as its letters would show, so the choice was between leaving the
page out and losing what the escape does; taro chose losing it, as long as
the build says so: "that way it can be played without solving that
problem. I'd leave a compiler warning or whatever so we don't lose and
forget it."

Until 2026-10-20 the colour, the speed and the instant text were dropped
too, and the window appeared with its box whole. One entry for the ink, and
not one per colour or a gradient, because the palette has no room and no
HDMA channel is free to rewrite an entry line by line: the window takes
three, the big picture's wave one, the face one, BG1's mask in a flattened
map one, and the other two are the general DMAs'. The ink costs the window
a colour and not the maps because a map that needs every entry merges its
own, on the screen, and the window's merge is the same in every box.

## How it applies

The rule is `escape_tail`, in `refit.rs` of `rm2k-snes-text`, with its
cases in that crate's `tests/escapes.json`; what each escape loses is said
by `with_wait_marks` there. The event compiler writes what is done as a
control byte —`CONTROL` in `messages.rs`— and notes the losses in
`Ctx::lose_dropped`, in `compile.rs`, with the text of `dropped_escape` in
`messages.rs`. The pace and the colour are `runtime/message_pace.s` in
the-advocate; the ink's colours, `inks` in `rm2k-snes-screens`'
`message_window.rs`. Tested by the test game's `test_escapes.py` —a box
with the gold window, a half space and an unknown letter shows its text,
and `degradations.json` lists each one— and `test_colours.py`: a box with
three colours, one half typed, and three speeds counted frame by frame.
Tasks:
[`unknown-message-escapes-are-dropped-with-a-warning`](https://github.com/lautarodragan/the-advocate-work/blob/main/tasks/unknown-message-escapes-are-dropped-with-a-warning.md)
and [`message-colours-and-speed`](https://github.com/lautarodragan/the-advocate-work/blob/main/tasks/message-colours-and-speed.md).
