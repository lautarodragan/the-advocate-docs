# The message box is the message window

## What

Desert Nightmare draws its message box with two pictures of its own,
`TextboxInnen` —a black panel— and `TextboxRahmen` —a double frame—,
320x84 at the bottom of the screen. Common event 2 fades them in before a
message, with the message transparent, and common event 3 fades them out.
On the console the box is **the message window itself**: the port's
manual patches fork both common events to turn the message's window on
and off instead, and replace the game's `Systemset_Spiel` with a copy
whose window frame is cut from the two pictures. Nothing in the tools is
specific to it.

It is taro's, from 2026-10-20: the frame becomes the game's window look,
its System graphic, so the console's own window draws it.

## Why

As pictures the box went as some 93 objects, up to 40 on a line where the
PPU draws 32, so while it faded in it showed as lines across the screen;
where it was drawn it went over the text, and on most maps it did not fit
at all and the text went over the map. The message window is on every map
already, in one frame, behind its own text, and costs no sprite tile.

## How it applies

The forks and how the System was made are told in
`desert-nightmare/patches/manual/README.md`, in the ports. What is lost:
the fade, 0.3 seconds each way; four lines of the box's height, since the
window is 80 high; and the box stays up only while a message does. The
menu's windows, which [ShowWindow](../../../decisions/show-window.md)
draws with the System in use, take the same frame. The task is
[`desert-nightmare-own-message-frame`](https://github.com/lautarodragan/the-advocate-work/blob/main/tasks/desert-nightmare-own-message-frame.md).
