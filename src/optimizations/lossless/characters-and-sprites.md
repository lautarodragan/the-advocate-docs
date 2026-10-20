# Lossless: characters and sprites

## Poses are uploaded on the fly

One pose per character lives in VRAM; the twelve poses of each one that
changes pose live in the cartridge, and the current one goes up by DMA in
vblank, as FF5 does. Still characters share a slot per (CharSet, index,
pose). An RM2k character is 192 object tiles with every pose resident,
and the PPU addresses 512.

- **Saves**: VRAM.
- **Costs**: DMA in vblank, with a cap per frame; when the queue is full
  an NPC waits, never the hero.
- **Where**: `vram_queue`, `vram_queue_hero` and `vram_drain` in
  `runtime/load_graphics.s`.
- **More**: [Poses are uploaded on the fly](characters-and-sprites/poses-on-the-fly.md).

## A pose goes up without its empty tiles

A pose is stored as a 16-bit mask of the 8x8 tiles that draw something,
and only those tiles. The slot remembers which of its tiles may hold
something, and the upload zeroes only those the old pose drew and the new
one does not.

- **Saves**: ROM —the padding of a 24x32 character in a 32x32 object, and
  the air above its head—; and fewer bytes per walking step in vblank.
- **Costs**: one word of bank 0's RAM per 32x32 block of the sprite bank.
- **Where**: `vram_drain` in `runtime/load_graphics.s`; `pose_record` in
  `rm2k-snes/mapdata/`.
- **More**: [Poses are uploaded on the fly](characters-and-sprites/poses-on-the-fly.md#and-a-pose-goes-up-without-its-empty-tiles).

## Distant still characters take turns on one slot

On a map that fits in no VRAM layout, and only there, still characters
that no camera position shows together share a slot even with different
poses, and the pose of the one that comes on screen is uploaded.

- **Saves**: sprite VRAM on the maps that would not fit otherwise.
- **Where**: `plan_sharing` in `rm2k-snes/wally/src/map.rs`, `share_table`
  in `rm2k-snes/mapdata/`, `runtime/shared_poses.s`.
- **More**: [The VRAM split](../../internals/vram.md#when-no-layout-is-enough-the-still-ones-take-turns).

## The sprite slots as a cache

On a map that does not fit even with turns, and only there, nobody owns a
slot: whoever comes on screen asks for one, and a slot not drawn for two
frames goes to the next. The exception: if the cache is full, the one
asking waits without being drawn until someone leaves the screen.

- **Saves**: sprite VRAM on crowded scenes.
- **Where**: `Mode::Cache` in `rm2k-snes/wally/src/map.rs`, `cache_table`
  in `rm2k-snes/mapdata/`, `runtime/cache_slots.s`.
- **More**: [The VRAM split](../../internals/vram.md#when-not-even-taking-turns-is-enough-a-slot-cache).

## The possible leaders share one CGRAM slot

Only one leads at a time, so giving each possible leader a palette of
its own would keep it taken by someone who is not on screen: they share
one, and the console uploads the colours of whoever leads. The exception
is a leader who is also an NPC on that map —Kersh on ABL's maps 1 and 6,
Talan on 3—, who gets their own, because both rows are in use at once.
Their own **while one is left for the other events**: Dooms 2's first map
shows every candidate of its character choice as an event, and they would
take all eight. The ones past that share a palette of the map like any
event, and their row of the actor table carries that palette's colours, so
the console's upload when they lead repaints nothing.

- **Saves**: sprite palettes.
- **Where**: `drawable_npcs` in `rm2k-snes/wally/src/sprites/split.rs`.
- **More**: [The possible leaders are an approximation](../../decisions/the-possible-leaders-are-an-approximation.md),
  who counts as one.

## The graphics only the hero shows go in the leaders' slot

A graphic only the hero ever shows is drawn like a possible leader: in
the leaders' CGRAM slot, with colours of its own that the console uploads
when the hero takes it. Since 2026-10-11. "Only the hero" means no move
route gives it to an event: the graphics of `ChangeSpriteAssociation`, and
those of move route command 34 on the hero; the event compiler's scan says
which graphics a route gives an event (`graphics_on_events`), and the
converter hands the rest to Wally as `hero_graphics`. One that an event of
the map also draws as a page graphic is left out: that one needs its
colours in the map's palette.

Before, they were split among each map's eight sprite palettes like any
other graphic of command 34, and two things came out of that, measured on
Vampires Dawn on 2026-10-11: the cartridge carried **a pose bank per map**
for each of them —a pose bank is the twelve poses as one map quantized
them—, and they were seen with the wrong colours in most maps. In the
leaders' slot the bank is the same in every map, and they take no palette
from the map. On that day's emission request of Vampires Dawn: 1,498 pose
banks to 1,163, and the maps with some graphic drawn with someone else's
palette from 365 to 77.

- **Saves**: ROM —a pose bank for the whole cartridge instead of one per
  map— and sprite palettes in every map; and the graphic is seen with its
  own colours.
- **Where**: `own` in Wally, and the colours of every graphic of command
  34 that ends in a leaders' palette; `hero_colors` and `route_graphic` in
  `rm2k-snes/mapdata/`; `route_gfx_colors`, `hero_poses_of_graphic` in
  `runtime/party.s`, and `graphic_hero`, which runs `apply_tint`.
- **More**: [Move route command 34 on the hero](../../decisions/move-route-command-34-on-the-hero.md).

## Finished routes give up their slot

The ROM has twelve slots for event routes. When none is free, the first
one whose route has finished goes to the event that asks: the cap is
twelve moving at once, not twelve on the whole map.

- **Saves**: bank 0 RAM, which is full.
- **Where**: `reclaim_slot` in `runtime/event_pages.s`.
- **More**: [Finished routes give up their slot](../../divergences/finished-routes-give-up-their-slot.md).

## Battle animations: one box for the whole ROM

A battle animation on the map is drawn in a box that is the same for
every map —a row of the sprite bank, four 32x32 objects—, so its frames
are converted once and every map that shows it shares them in the
cartridge. What the box costs in fidelity is in
[Lossy](../lossy/pictures-and-animations.md#battle-animations-on-the-map-shrunk-to-a-box).

- **Saves**: ROM —one copy of each animation's frames, not one per map—.
- **Where**: `rm2k-snes/convert/src/animations.rs`.
- **More**: [The shrunk battle animation](../../divergences/the-shrunk-battle-animation.md).

## A battle animation's frame: without its empty tiles, one at a time

Each frame of a battle animation is stored as the list of the box's tiles
that draw something, in LZ2, and is decompressed when it is due, not the
whole animation when it starts. The box's 2 KB buffer is kept at zero but
for the last frame's tiles, so building the next one clears those and
copies the new ones, unrolled; the vblank uploads the 2 KB whole.

- **Saves**: ROM —the frames are mostly transparency—, the frames a
  whole animation would stop the map for while it decompresses, and CPU
  per frame built.
- **Where**: `frame_without_gaps` in `rm2k-snes/mapdata/src/emit.rs`;
  `anim_frame_build` and `anim_buf_clear` in
  `runtime/battle_animation_frames.s`.
- **More**: the header of `runtime/battle_animation_frames.s`.

## Still events that animate are walked only when a column changes

The still events that run through their walk cycle take their column from
the game tick, so the rows are walked only on the tick where the soonest
of them changes column, or when something else wrote one of their poses;
the frames in between cost a subtraction.

- **Saves**: CPU on maps with many of them.
- **Where**: `animate_still` and `still_touched` in
  `runtime/animated_stills.s`.
- **More**: the header of `runtime/animated_stills.s`.

## The timer and the raindrops fill the last row's free columns

The timer's glyphs and the raindrops are a strip two tiles tall in the
sprite bank, and it goes into the columns the last row of blocks leaves
free when they are enough, instead of starting a row of its own.

- **Saves**: sprite VRAM —on a map with a slot cache, slots—.
- **Where**: `strip_base` in `rm2k-snes/wally/src/sprites/extras.rs`.
- **More**: [The VRAM split](../../internals/vram.md#the-timer-and-the-raindrops).

## A small picture: its sparse blocks in 8x8, its repeated tiles once

A `ShowPicture` picture in sprites is cut into 32x32 objects, but a block
with four of its sixteen 8x8 cells drawn or fewer goes as 8x8 objects
whose tiles fill what the last row of blocks leaves free, and the same
pixels take their tiles once, whether in one picture or in two. Since
2026-10-20 a thin block —up to eight cells in at most two of its rows, not
two whole rows side by side— goes as cells too: it saves its tiles, and the
slivers a 32x32 costs on the lines where it draws nothing.

- **Saves**: sprite VRAM.
- **Where**: `Bank::cut` and `as_cells` in
  `rm2k-snes/convert/src/pictures.rs`; `picture_tables` in
  `rm2k-snes/mapdata/src/emit.rs`; `pic_draw` in `runtime/picture_far.s`.
- **More**: [The VRAM split](../../internals/vram.md#the-small-pictures-go-at-the-end-of-the-sprite-bank).
