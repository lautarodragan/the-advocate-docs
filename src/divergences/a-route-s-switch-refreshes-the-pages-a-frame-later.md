# A route's switch refreshes the pages a frame later

## What

Route commands 32 and 33 —`switch_on` and `switch_off`— turn a switch on
or off in the middle of a route, forced (`MoveEvent`) or the page's own.
In RM2k, `Game_Character::UpdateMoveRoute` sets the switch and calls
`Game_Map::Refresh()` **on the spot**: every event takes its new page
before the route's next command runs. If the walking event itself got a
new page, its own route starts over and that frame's commands stop there.

Here the switch is set on the spot too, with the same routine a
`ControlSwitches` uses —`set_switch`, in `runtime/variables.s` in
the-advocate—, and the refresh is the one that routine asks for:

* the pass over the **autostart** pages runs on the same frame, on leaving
  the interpreter —the routes run before it, see `route_switch` in
  `runtime/event_routes.s`—, so a page that the switch starts, starts when
  it would in RM2k;
* the **full** refresh —which page each event shows, its graphic, its own
  route— runs at the start of the next frame.

So what a route's switch changes on screen shows **one frame late**, and a
route whose switch changes its own event's page runs the rest of that
frame's commands —the turns and the instant ones after it— before its new
page's route replaces it. The step the hero takes on that frame, and what
he touches, still see the pages from before the switch.

## Why

It is the same refresh `ControlSwitches` has since the interpreter stopped
refreshing before each command: the full pass looks at every page of the
map, and running it in the middle of the routes' pass would cost the frame
where several land together. Nothing measured asks for more: the switches
in the eight games' routes are at the end of a route or turn on a
condition another event's page reads, and a frame later is not seen.

## When it stops being enough

A route that turns a switch and goes on with commands that depend on the
new pages on the same frame —a step onto a cell that the switch frees—,
or one that changes its own page and relies on the restart cutting the
commands after the switch.
