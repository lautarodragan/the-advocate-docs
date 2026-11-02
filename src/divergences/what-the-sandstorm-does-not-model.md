# What the sandstorm does not model

## What

`WeatherEffects` type 4, the sandstorm, is drawn since 2026-11-02 —
`runtime/sandstorm.s` in the-advocate, which assembles only in a cartridge
where some page starts one—. RM2k's (easyrpg's `Game_Screen::UpdateSandstorm`
and `Weather::DrawSandstorm`) is two things, and here each goes its own
way:

* **The overlay**: a 16x16 tile of three sand colours pasted twice over the
  whole screen, at opacities that together cover 34%, 49% or 77% of it by
  strength, over the map, its upper layer and every character and under the
  flash, the pictures, the windows and the timer, tinted by the screen's
  tone. That is the set of colours the tint already transforms, so here it
  is one more step of the tint's channel tables: each channel `i` goes to
  `i * (1 - a) + tone(sand) * a`. It costs nothing per frame and takes no
  layer and no colour math.
* **The particles**: 128, 191 or 255 of 1x2 pixels, yellow, orange, red and
  white, born on a half ellipse below the top-centre and accelerating
  outward. Here they are 8x8 objects with their two pixels in the corner, at
  the end of the OAM like the rain's drops, and only the ones on screen are
  simulated.

What is lost:

* **The overlay is flat and still.** RM2k's tile has a grain of three
  colours, one step apart at five bits, and its two layers drift and bob;
  here it is their average, (230, 230, 170), and it does not move.
* **The grains are opaque.** RM2k's start at alpha 180 and grow to opaque;
  the console has no transparency per object.
* **At most 48 grains on screen.** At the strongest RM2k has 38 on average
  and up to 68: with 48 slots, 1.4% of the grains are not drawn (simulated
  in Python on RM2k's rule, at 256x224).
* **How many are born is a count, not a draw per particle.** The ones off
  screen are a ring of how many come back to life on each of the next 81
  frames and a pool of those waiting, and each frame `pool * 26 / 256` are
  born, rounded at random: the expected number of RM2k's ten percent each,
  without its spread.
* **A new sandstorm always starts as RM2k's first one**: its particles come
  alive over forty frames, a burst that comes back in waves. RM2k's later
  ones go on from where the previous storm froze them.
* **A grain whose eighty frames end goes away**; RM2k leaves it drawn where
  it stopped until it is born again. Only the ones born almost still are on
  screen by then.
* **The tone's saturation is not applied to the sand or to the grains**:
  only the channel tables are. With the saturation at 100 it is exact.
* **The grains go only where the converter thinks the storm can blow** —the
  map a page starts it on if the page stays there, and the ones a
  `Teleport` takes it to while it blows: `sandstorm_reach` in
  rm2k-snes-convert—, and on those, only if a sprite palette that is not the
  leaders' has four free entries or there is room for one more. Elsewhere
  the overlay is drawn without grains, and the converter warns with
  `sand-grains-not-drawn`.
* **The backdrop is not covered**: the overlay goes over the colours the
  tint transforms, and the screen's backdrop is not one of them, so a map
  smaller than the screen keeps its black border.

## Measured

On 2026-11-02, with Zounds at the-advocate's `tijereta-sandstorm`: the
grains go on maps 14, 50, 88, 111, 112 and 113, all with room, and no
other map of the game lost anything to them. In the release cartridge they
cost 24 bytes of bank 0, 1,668 of `$C1` and 527 of far RAM. On 113, after the
debug room's maze door, the ROM keeps 36 grains on screen on average over
400 frames, and RM2k's rule, simulated, 38.

## How it looks today

The Death Desert is pale sand with sparks of colour bursting from the top
of the screen, as in RM2k; the screen does not shimmer. ABL, Vampires Dawn
and every game without a sandstorm are byte for byte what they were.

## When it stops being enough

If the flat overlay looks dead next to RM2k's, the grain can come back as
a dithered pattern in a palette blend, or as colour math over a layer the
map does not use —in mode 3 there is none to spare, see
[Mode 3](../internals/mode-3.md)—. A map with characters filling every
palette loses its grains; the way out would be to let them borrow the
nearest colours, as the rain's drops do.
