# The MSU-1 is required, for now

## What

**Desert Nightmare requires the MSU-1**, as Vampires Dawn does: its project
file says `[rom] msu = "required"`, what does not fit in the ROM travels in
its `.msu`, and without it the cartridge shows a screen that says so and
does not start. It is the exception [the MSU-1 is required only when the ROM
does not reach](../../../decisions/the-msu-1-is-required-only-when-the-rom-does-not-reach.md)
allows, written down here because it is this game's.

What it pays for is **the towns painted**: every map whose still pieces are
painted into its panorama (`[graphics] painted_events`, see [the VRAM
split](../../../internals/vram.md#the-still-pieces-painted-in)) carries each
8x8 cell of itself in the ROM, and painting every map that benefits does
not fit beside the rest. The cells stay in the ROM —they are uploaded column
by column while the camera moves, and the `.msu` takes only what is read
when a map or a battle loads—, and the packing sends that to the `.msu`
instead: the maps' streams, the panoramas' tiles, the samples.

## Why

taro, on 2026-11-19, asked whether storing each painted cell once was the
way, translated:

> If it's impossible to make a game run without the .msu, we make the .msu
> mandatory. Vampires already works that way. I think it's important that
> the .msu be optional where we can (ABL, mainly), but it isn't worth
> looking for a very complex path to satisfy that need. If we go this way,
> let's decide that the msu will be mandatory for Desert Nightmare for
> now, to reduce the challenge.

The other way —each painted map's cells stored once, gathered by an index
when a column is uploaded— needs the runtime's upload rewritten and its
vblank time measured, for a game whose story does not get past its first
day yet. The research that measured both is [Can the towns be
baked?](../research/2026-11-19-can-the-towns-be-baked.md).

## How it applies

- **For now.** When the painted cells are stored once —the draft
  [`painted-maps-carry-every-cell`](https://github.com/lautarodragan/the-advocate-work/blob/main/tasks/painted-maps-carry-every-cell.md)—
  the game should go back to `"optional"` if its ROM reaches.
- **Nothing of this is Desert Nightmare's code**: the knob and the packing
  are every game's, and A Blurred Line stays without the MSU-1 required.
