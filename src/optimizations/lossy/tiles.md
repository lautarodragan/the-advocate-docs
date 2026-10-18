# Lossy: tiles

## Similar background tiles are fused, as a last resort

A map with more distinct tiles than any VRAM layout holds has its most
similar tiles fused until it fits, only after everything else that could
make room.

- **Loses**: detail where two tiles were almost the same.
- **Where**: `fuse_lower` in `rm2k-snes/wally/src/background/fuse.rs`,
  over Adeleine's `snes/adeleine/src/fuse.rs`.
- **More**: [The VRAM split](../../internals/vram.md).

## A window with a gradient is fused until it fits

A System whose stretched background is a gradient is a different cell on
every row of every window. The message window fuses the background seen
through its frame into bands; the screens that cut their windows into a
bank merge their least visible cells one at a time until the bank holds
them; End Game merges colours.

- **Loses**: the gradient goes in bands, first where it shows least.
- **Where**: `fit_frame` in `rm2k-snes/screens/src/message_window.rs`,
  `fit.rs` and `end_game.rs` in `rm2k-snes/screens/`; Adeleine's
  `group_tiles`, `merge_least_visible_tiles` and `merge_until`.
- **More**: [A window with a gradient, fused until it fits](tiles/a-window-with-a-gradient.md).

## The windows' background, a colour of the project's

A game can fill a System's windows with one colour —or a vertical
gradient— of its own: each window's inside is then one cell and one CGRAM
entry, and nothing has to be fused. It is the game's choice, per System.

- **Loses**: what the System's background had besides its colours —a
  dither, a stroke near the edges—; a gradient is drawn as its average
  until the runtime draws gradients.
- **Where**: `windows.rs` in `rm2k-snes-project`; `--window-background` in
  `rm2k-snes-screens`.
- **More**: [The windows' background is the project's, by System](../../decisions/the-windows-background-is-the-projects-by-system.md),
  [The windows, made cheaper](../../research/2026-10-11-the-windows-made-cheaper.md).

## A panorama's similar tiles are grouped

A panorama brings far more distinct tiles than its layer has room for, so
similar ones are drawn with a single one; at 4 or 2 bits its colours are
merged too.

- **Loses**: the grain, slightly; at 2 bits, gradients flatten.
- **Where**: `group_tiles` in `snes/adeleine/src/tiles.rs`, used by
  `rm2k-snes/wally/src/background/panorama.rs`.
- **More**: [Panoramas](../../research/2026-10-05-panoramas.md).

## A map that shows its panorama gives up its layer above

In mode 3 the panorama takes BG2, which is where the layer above the hero
would go. A map that does draw something above the hero has it
**flattened**: what it draws there goes into the background, and the 8x8
cells it covers at least half of go to BG1's priority 1, in front of the
hero. A map that would put too much on the wrong side of the hero that way
goes in mode 1 instead: the map at 4 bits in BG1 and BG2 and the panorama
at 2 bits in BG3.

- **Loses**: flattened, the hero's depth against what is above him is
  decided per 8x8 cell, not per pixel; in mode 1, the map's colours go to
  4 bits per tile, and the panorama's to 2.
- **Where**: `Way::Flattened` and `Way::Mode1` in
  `rm2k-snes/wally/src/background/panorama.rs`; `ABOVE_LOST_MAX` in
  `rm2k-snes/wally/src/layers.rs`.
- **More**: [Mode 3 by default](../../internals/mode-3.md),
  [Panoramas](../../research/2026-10-05-panoramas.md).

## A ChipSet animation that does not fit in WRAM stands still in part

The drawings of the water and the animated tiles have a fixed WRAM area;
a map whose drawings take more keeps moving the tiles that cover the most
cells, and the rest stay on their first drawing.

- **Loses**: some water cells stand still, mostly at the edges.
- **Where**: `rm2k-snes/mapdata/src/animation.rs`; `ANIM_DRAWINGS` in
  `runtime/far_ram.s`.
- **More**: [A ChipSet animation that does not fit in WRAM](../../divergences/a-chipset-animation-that-does-not-fit-in-wram.md).

## The ChipSet animation's clocks come from the game tick

The water and the animated tiles keep no counter and no pointer of their
own: where each is in its cycle comes from dividing the sixteen-bit game
tick by its rate, with the hardware's divider. Neither rate divides 65,536,
so the cycle skips a frame when the tick wraps around.

- **Loses**: one frame of the water every 65,536 ticks, about eighteen
  minutes of play.
- **Saves**: bank 0's RAM, which had no byte left for them.
- **Where**: `anim_tick` in `runtime/chipset_animation.s`.
- **More**: [The ChipSet skips a frame every 18 minutes](../../divergences/the-chipset-skips-a-frame-every-18-minutes.md).

## The choice bar keeps one of its two drawings

The System carries two drawings of the cursor and RM2k alternates them;
the bar is composed into the window's tiles like the text, so the console
draws only the first one.

- **Loses**: the cursor's blink between its two drawings.
- **Saves**: recomposing the highlighted line every ten frames, or the
  tiles of a second version of it in VRAM.
- **Where**: `msg_bar_source` in `runtime/message_arrow.s`; `Skin::cursor`
  in `rm2k-snes/screens/`.
- **More**: [The choice bar does not alternate its two drawings](../../divergences/the-choice-bar-does-not-alternate-its-two-drawings.md).
