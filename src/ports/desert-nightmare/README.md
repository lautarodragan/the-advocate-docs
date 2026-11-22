# Desert Nightmare

## Known issues

What playing the port shows today that the original does not. It is kept
current: whoever fixes one removes its line in the same change. The work
behind each one is a draft or a task in the tasks repository, named here
as text.

* **The menu is windows of the System, not the original's pictures**:
  text where the original draws, a name where it has an icon —cut to its
  square: "Room No. 4" for the room key— and the full name in the box
  below. See [ShowWindow](../../decisions/show-window.md).
* **After a note or a map is put away, the menu stands over the room
  instead of the burning face**, until it is closed and opened again.
  Draft: [`desert-nightmare-the-menu-loses-its-face-after-a-note`](https://github.com/lautarodragan/the-advocate-work/blob/main/tasks/desert-nightmare-the-menu-loses-its-face-after-a-note.md).
* **On most maps the menu's burning face goes black past its first
  screen**: behind the options it shows, a little off; behind the
  inventory, the keys, the notes and the maps there is black, with a
  sliver of the face at the right edge. On every painted map —the towns,
  the one-screen interiors—. Draft:
  [`desert-nightmare-the-menu-s-face-is-lost-on-painted-maps`](https://github.com/lautarodragan/the-advocate-work/blob/main/tasks/desert-nightmare-the-menu-s-face-is-lost-on-painted-maps.md).
* **A map from the menu shows no mark of where Sandra is** on some maps:
  the mark is a small picture, and the map she stands on has no room for
  it. Draft: [`desert-nightmare-the-map-cursor-is-shared-on-ten-maps`](https://github.com/lautarodragan/the-advocate-work/blob/main/tasks/desert-nightmare-the-map-cursor-is-shared-on-ten-maps.md).
* **The menu's windows go away while a box is up** —"You have no items.",
  "Really quit?"— and come back when it closes; the original keeps its
  pictures under the box. See [ShowWindow](../../decisions/show-window.md).
* **The message box does not fade**: it is up at once and gone at once,
  where the original fades it in and out, and it closes and opens again
  between two messages with something else in between —a wait, a sound—,
  where the original keeps it up. It is 4 lines shorter, too. The box is
  the console's message window with the game's frame; the menu's windows
  have that frame as well. See [The message box is the message
  window](decisions/the-message-box-is-the-message-window.md).
* **Where nine or more characters stand on one line, the ones further
  back are cut** —a column missing, or gone—: the console draws 34
  slivers of 8 pixels a line and each character takes four. On 7 maps:
  the apartment at night (33, 71), the exit's walking crowd (35, which
  also plays slower), the post office (44), 47 and the mines (48, 102).
  Draft: [`desert-nightmare-characters-cut-on-crowded-lines`](https://github.com/lautarodragan/the-advocate-work/blob/main/tasks/desert-nightmare-characters-cut-on-crowded-lines.md).
* **Sandra's name and the other coloured words have fewer shades** than
  RM2k's gradient, and the message window one colour less: its last entry
  is the box's ink. See [What message escapes do not model](../../divergences/what-message-escapes-do-not-model.md#the-colour).
* **Some doors, lights and people show up late, or not at all,** on 7
  maps —the first street (2), the exit (35), the mines (102, 105), the
  workshop's cellar and the vault (137, 138), the old mine settlement
  (172)—: more of them
  stand on screen at once than the sprite cache has slots, and the last
  ones wait for one. Draft: [`desert-nightmare-events-wait-for-a-slot`](https://github.com/lautarodragan/the-advocate-work/blob/main/tasks/desert-nightmare-events-wait-for-a-slot.md).
* **At night the town has no lamps and no lit windows** on the eight
  night maps that scroll —the entrance (59), the workshop's street (61),
  the houses (62), the police station (66), the post office (68) and the
  motel at night (32, 69, 70)—: the night is a tint, dark everywhere, and
  a character walking under a lamp stays dark. Task:
  [`desert-nightmare-lit-cells-over-a-tinted-night`](https://github.com/lautarodragan/the-advocate-work/blob/main/tasks/desert-nightmare-lit-cells-over-a-tinted-night.md).
* **The motel's rooms, the warehouse, the dream, the mines and the
  workshop's cellar are lit as by day**, without their darkness and their
  lamps' light, on 17 maps (16, 17, 22, 48, 58, 102 to 106, 136 to 138,
  143, 144, 157, 182): their light masks are placed by a variable and not
  drawn. Draft:
  [`desert-nightmare-light-masks-placed-by-an-anchor`](https://github.com/lautarodragan/the-advocate-work/blob/main/tasks/desert-nightmare-light-masks-placed-by-an-anchor.md).
* **The characters are not shaded by the lights.** Sandra stays at her
  full colours in the WC's darkness and in every lit interior: the masks
  are drawn into the map, not over her. See [Pictures drawn into the map](../../divergences/pictures-drawn-into-the-map.md).
* **The menu and the other screens keep the intro's System** for the whole
  game: the original switches to `Systemset_Spiel` in its common event
  `Init`, and only the message window follows. See [What `ChangeSystemGraphics` does not model](../../divergences/what-changesystemgraphics-does-not-model.md).
* **The hit dog shows on the sand beside the warehouse's roof** in the
  scene where the dog seems to die (11), a little lower, slides behind the
  roof and leaves a sliver of itself at its edge; the original hides it
  behind the roof. Draft:
  [`desert-nightmare-the-hit-dog-lands-beside-the-roof`](https://github.com/lautarodragan/the-advocate-work/blob/main/tasks/desert-nightmare-the-hit-dog-lands-beside-the-roof.md).
* **The warehouse's first room (21) is darker** than the original's once
  its opening scene has played: the light drawn into it is the one the
  scene leaves, not the one the room shows each time it is entered. Draft:
  [`desert-nightmare-the-warehouse-is-darker`](https://github.com/lautarodragan/the-advocate-work/blob/main/tasks/desert-nightmare-the-warehouse-is-darker.md).
* **The safe's and the shrine's locks do not open their screen** (maps
  115 and 161): their symbols stand frozen in the room from the start,
  and looking at the lock shows its instruction and nothing more, where
  the original opens a plate with four wheels that turn. Draft:
  [`desert-nightmare-the-locks-pictures-are-drawn-into-their-rooms`](https://github.com/lautarodragan/the-advocate-work/blob/main/tasks/desert-nightmare-the-locks-pictures-are-drawn-into-their-rooms.md).
* **Two apartments keep their first background** where the original
  changes it: the dream one on map 34 and the one on map 82. Task:
  [`a-panorama-with-pictures-drawn-into-it-is-not-swapped`](https://github.com/lautarodragan/the-advocate-work/blob/main/tasks/a-panorama-with-pictures-drawn-into-it-is-not-swapped.md).
* **The warehouse's monster is not seen coming**: on its last room (28)
  the room stays still for six seconds before the monster appears, where
  the original shows its silhouette coming down from the roof; on its
  first room (21) the shadow that crosses it is missing. The rooms have
  no sprite tiles to spare for them. Draft:
  [`desert-nightmare-the-warehouse-monster-is-not-drawn`](https://github.com/lautarodragan/the-advocate-work/blob/main/tasks/desert-nightmare-the-warehouse-monster-is-not-drawn.md).

## The towns, painted (2026-11-19)

The towns are built out of events, a 24x32 piece of a building per cell
over a panorama with white holes, and the console drew each as an object:
white rectangles where its slots, its table or its RAM ran out, and two
frames a tick on 18 maps. Since this day the project has `[graphics]
painted_events` on 95 maps: the pieces that never change are painted into
the panorama with the ChipSet's layers, and the maps go painted —see [the
still pieces, painted in](../../internals/vram.md#the-still-pieces-painted-in)—.
The towns play at a frame a tick with their buildings whole, and the
captures against EasyRPG are named in the task
[`still-pieces-painted-into-a-painted-map`](https://github.com/lautarodragan/the-advocate-work/blob/main/tasks/still-pieces-painted-into-a-painted-map.md).
The painted maps required the MSU-1 for a day, until each one carried each
distinct cell once: [the MSU-1 is required, for now](decisions/the-msu-1-is-required-for-now.md),
undone on 2026-11-20.
Which maps, and why not the others, is said by the project's
`advocate.toml`.

On 2026-11-22 the 42 one-screen interiors went painted too: they had been
left out because the message box's two pictures did not fit beside a
painting, and since the box is the message window those are never shown.
Their furniture, which as characters stood lit over the room's light, is
painted under it, as RPG_RT draws it. Task:
[`desert-nightmare-interiors-painted-now-that-the-box-is-a-window`](https://github.com/lautarodragan/the-advocate-work/blob/main/tasks/desert-nightmare-interiors-painted-now-that-the-box-is-a-window.md).

## The nights that scroll, tinted (2026-11-19)

The game's night is two pictures the size of the map, a dark mask with the
lamps' cones and a canvas with the lit windows. On the eight night maps
that scroll they are placed at an anchor event's screen position, which the
converter does not draw, and the town showed as day. Since this day the
port's manual patches turn the mask into an RM2k `TintScreen` on those
maps, `desert-nightmare/patches/manual/` in the ports: the console comes
within 0.1 to 0.3 of 31 of EasyRPG running the same patches, and the lamps
and windows are what is left. Why, the fit, and the ways of lighting a
tinted night, measured: [Nights with a tint of their own](research/2026-11-19-nights-with-a-tint-of-their-own.md).

## What it takes to be playable (2026-11-19)

Played through the harness as far as it goes —the middle of day 1— and
compared with RPG_RT, what breaks, why, and what to do first: [What it
takes to be playable](research/2026-11-19-what-it-takes-to-be-playable.md).

## The lights, drawn into the map (2026-10-16)

The game lights its interiors and its nights with a `ShowPicture` per map:
~150 `Lichteffekt_*` masks the size of the map, fixed to it, at
transparency 20 to 65 —most 35 to 45—, a near-black with the lamps' cones
in a gradient, never erased while one is there. Each was a big picture on
the console, with the map hidden under it: the WC (map 7) went dark for the
whole scene. Since this day the project has `[graphics] map_pictures`, and
they are drawn into the map's tiles at their RM2k opacity, the layer above
the hero included: 155 pictures in 126 of the 189 maps, in the build's
log, map by map ("drawn into the map"). On the WC, the console's capture
against RPG_RT 2015 under Wine differs by 0.35 of 31 levels at 15 bits over
the mask; what the console's colour math would have given instead —a 50 %
blend, a subtraction— is five times farther, and there is no layer free
for it in mode 3 anyway. The measurement and the captures are in the task
[`pictures-that-go-over-the-map-outside-sunset`](https://github.com/lautarodragan/the-advocate-work/blob/main/tasks/pictures-that-go-over-the-map-outside-sunset.md);
what is lost, in [Pictures drawn into the map](../../divergences/pictures-drawn-into-the-map.md).
The translucent ones the rule does not take —the flashlight on 82, which a
`MovePicture` moves; `GO_Text` and the intro's texts, fading over the
screen— stay big ones, listed in `degradations.json` as
`translucent-picture-hides-the-map`.

## What it leaves out (2026-10-11)

No map: the whole release builds. What stopped twelve maps on 2026-10-10
degrades now, with a warning: past 120 events the ones left out lose only
their looks or an invisible wall, past 104 drawn events the rest aren't
drawn and still run, and the panoramas are drawn flattened or in mode 1. None of those
maps was looked at in an emulator.

Since later on 2026-10-11 it builds with its own System: End Game's
colours are merged until its pairs fit the palette —[A window with a
gradient is fused until it fits](../../optimizations/lossy/tiles/a-window-with-a-gradient.md)—.

## The empty party (2026-10-10)

The `party = [1]` crutch is gone: the game starts with the party its
database gives, which is nobody, as in RM2k. The hero isn't drawn until
someone joins, and still walks, collides and touches events; the pause
menu grays Items and Equipment while the party is empty. Built without
the crutch, the whole release builds.

## As of 2026-10-10

> Measured again on 2026-10-10 ([Other games, measured again](../../research/2026-10-10-other-games-measured-again.md),
> with the tools of the-advocate `1afe6457` and the ports at `951471f`),
> without fixing anything. What follows the next heading is the snapshot
> of 2026-10-06, kept as it was.

- **The cartridge builds** with A Blurred Line's System and
  `party = [1]`. With its own System, "the End Game screen uses 18 pairs of
  colours and a 4bpp palette has 16"; with an empty party, "the party is
  empty".
- **It needs twelve maps out, not twenty-four**: the six panoramas without
  a VRAM layout, the three with more than 104 NPCs, the two with more than
  120 events and map 182's tile animation. The committed project still
  leaves out the survey's twenty-four.
- **Each map**, next to the start map: 177 of 189, the same twelve.
- **The pages that compile**: 7,277 of 7,340 on the maps (7,150 on
  2026-10-06), 28 of 30 common events, 0 of 100 troop pages. What is left
  is mostly `ControlVars` reading an event's screen position.
- **What degrades**: 136 maps don't draw some of their pictures.

## As of 2026-10-06

> **A snapshot as of 2026-10-06.** What the survey of 2026-10-05 found
> ([Other games: the survey](../../research/2026-10-05-other-games.md),
> measured on `24e0223e` of the-advocate) and what the tasks that followed it
> changed up to 2026-10-06. It doesn't describe the port after that date.

The project is `desert-nightmare/` in the-advocate-ports, without MSU-1 and
without tests (see [Ports](../README.md)).

## The game

189 maps, in English. It has no battles, and it starts with an empty
party. Its music is MP3, through Ineluki's Harmony patch v1.1.1 with
links: fifteen text `.link.wav` files (`..\mp3\Credits.mp3` and `loop`)
that its Harmony turns into an MP3 track. The port doesn't read them:
those themes don't play. It is the only thing from outside RM2k that
changes what is heard, and the MSU-1 would be the natural way to carry it;
without MSU-1, it would take getting a MIDI or silencing them. No DynRPG,
no custom fonts, an RM2k database.

## What the survey found (2026-10-05)

- **Crutches**: the RTP's System reduced to 12 colors, in a copy of the
  game, because its own didn't fit even reduced; `party = [1]`, because
  RM2k starts with nobody and no visible hero and the converter asks for a
  leader to draw; and no title screen, because its title doesn't fit in
  VRAM next to the menu.
- **Each map alone**: 177 of the 189 build a cartridge; panoramas with no
  VRAM layout cut 6, more than 104 NPCs in OAM cut 3, more than 120 events
  cut 2, and the tile animation in WRAM cuts 1.
- **The cartridge**: starting on map 1, 165 of the 189; out are 24: the
  ones of the sweep, and twelve more for `ROUTE_GFX_MAX` with neighbors.
- **The pages that compile**: 5,190 of 7,340 on the maps, 24 of 30 common
  events, 0 of 100 troop pages. What blocks them: `\C[n]` (2,144 boxes),
  the faces (2,049 uses of `ChangeFaceGraphic`), `ConditionalBranch` on an
  event's direction (354), `EraseScreen`/`ShowScreen` (698), `EraseEvent`
  (157 pages, 127 of them only for that), `ChangeSystemGraphics` and
  `InputNumber` with a variable.
- **Next to the original**: the picture menu ("With Intro / No Intro /
  Quit") comes out right, but **the background, a panorama in mode 3, is
  light-blue garbage**. It was a bug of the panoramas, merged that same
  day, in no golden because A Blurred Line has no such panorama; the
  survey put it first, before anything else.
- **Fixed in the survey's branch**: the fallback battle backdrop is asked
  for only if some battle needs it.

## What changed after it (to 2026-10-06)

- **The window**: its System's 15 colors fit; what stopped it was End
  Game, another limit. (Task [`window-with-more-than-15-colors`](https://github.com/lautarodragan/the-advocate-work/blob/main/tasks/window-with-more-than-15-colors.md).)
- **Choosing another System from the project**: built with the original
  game and `system = "ablskin.png"` (A Blurred Line's System), it starts.
  (Task [`choose-another-system-from-the-project`](https://github.com/lautarodragan/the-advocate-work/blob/main/tasks/choose-another-system-from-the-project.md).)
- **Color codes ignored**, as a debt (`\c[1]Sandra: \c[0]` in almost every
  message, 1,345 pages out): map pages that compile went from 5,738 to
  7,034 of 7,340. (Task [`ignore-the-color-codes-in-messages-for-now`](https://github.com/lautarodragan/the-advocate-work/blob/main/tasks/ignore-the-color-codes-in-messages-for-now.md).)
- **`EraseEvent` for real**: it was what kept the most pages out alone, and
  it no longer does; the 157 are parallel pages without a graphic. It was
  suspected of freezing the intro, and it wasn't that: the intro ends in
  black on the street (map 9) because its last page teleports to map 2,
  which the survey left out. What did change is the WC scene (map 7): a
  picture that RM2k shows as a dark mask at 35 % comes out opaque and
  covers the room for ~480 frames —it looks frozen, but it goes on—. That
  is the pictures' translucency, not the erasing. (Task
  [`eraseevent-for-real`](https://github.com/lautarodragan/the-advocate-work/blob/main/tasks/eraseevent-for-real.md).)
- **`EraseScreen`/`ShowScreen` accepted without doing anything**, as a
  debt: the map stays visible where the original goes black, and the start
  menu comes out over VRAM garbage. The title that doesn't fit now
  degrades: it builds without `--no-title`. The survey's exclusions stay:
  it wasn't tested which depended on `EraseScreen`. (Task
  [`degrade-instead-of-failing`](https://github.com/lautarodragan/the-advocate-work/blob/main/tasks/degrade-instead-of-failing.md).)
- **Four shortcuts**: the condition on direction (21 pages) and the random
  step (order 8, 39 pages). Map pages went from 7,034 to 7,150 of 7,340.
  (Task [`four-shortcuts-for-the-other-games`](https://github.com/lautarodragan/the-advocate-work/blob/main/tasks/four-shortcuts-for-the-other-games.md).)
- **Decorative events as background**: 2,887 candidates, measured without
  looking at the drawing. (Task [`decorative-events-into-the-background`](https://github.com/lautarodragan/the-advocate-work/blob/main/tasks/decorative-events-into-the-background.md).)
- **A ChipSet animation that doesn't fit in WRAM** (2026-10-10) keeps the
  tiles that cover the most cells moving and leaves the rest still: in map
  182, 9 of its 51 water tiles (44 of 1,469 cells). The map is back in the
  cartridge, which builds whole. (Task
  [`tile-animations-that-dont-fit-in-wram`](https://github.com/lautarodragan/the-advocate-work/blob/main/tasks/tile-animations-that-dont-fit-in-wram.md).)

The tasks named here are in `tasks/testing/` in the-advocate-work.
