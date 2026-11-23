# RAM is placed by the linker

## What

**A new variable of the runtime gets no address written by hand, and is
declared in the file of the subsystem that owns it**, with a `.res` in the
segment of its RAM:

```
.pushseg
.segment "FARBSS"        ; or "BSS", or "ZEROPAGE"
my_state: .res 2
.popseg
```

and `ld65` places it. For bank 0 —`ZEROPAGE` and `BSS`— that was already
how addresses were given; what is new is that the declaration leaves the
three global files of `runtime/` in the-advocate (`zero_page.s`,
`low_ram.s`, `far_ram.s`) for the subsystem's own. For far RAM, which
until then was a map of addresses written by hand, it is a new segment,
`FARBSS`, in an area of the `.cfg` that `rm2k-snes-cartridges assemble` writes
(`FAR_POOL_START` in `rm2k-snes/cartridge/src/banks.rs`).

**Since 2026-11-19 it holds for the whole WRAM**: the grid's halves, the
load buffer, `GRID_TOP` and the stack are areas of the `.cfg` too, and
nothing in the runtime is at an address written by hand. How the 128 KB
are laid out, [The WRAM map](../internals/wram.md). Decided on
2026-10-11, in the task
[`global-variables-collide`](https://github.com/lautarodragan/the-advocate-work/blob/main/tasks/global-variables-collide.md),
for new variables; taken to the rest of the WRAM in
[`all-wram-placed-by-the-linker`](https://github.com/lautarodragan/the-advocate-work/blob/main/tasks/all-wram-placed-by-the-linker.md),
after taro asked on 2026-11-16 why anything was still placed by hand.

## Why

Two different collisions, and the second one was the dangerous one:

* **Two branches that each add a variable collided at the end of the same
  file**, even when they worked on subsystems that never touch: the split
  of `main.s` spread the code by subsystem, but not the memory. A
  declaration in the subsystem's file does not touch any line another
  branch touches.
* **Far RAM addresses were chosen as "the next free one"**, looking at
  each one's copy. Two branches could choose the same address and git saw
  nothing; the merge assembled and ran. It happened more than once —the
  picture on top of the MSU-1 state, which silenced the music; a call
  stack set aside inside the parallel interpreters' list—, and the
  workaround was handing out ranges in advance to every agent. With the
  linker, two variables in `FARBSS` cannot overlap by construction, and
  when the area is full the link fails instead of overwriting.

Leaving padding lines per subsystem, or keeping the files in alphabetical
order, would have spread the textual collisions, and left the far
addresses chosen by hand.

## How it applies

* **`FARBSS` is declared `far`**, so its labels carry bank `$7E`: read
  plainly, they are a long access, right whatever the data bank. Code that
  runs with the data bank at `$7E` takes `.loword(name)`, and so does a
  line of `save_table`, which is a `.word`. A label used **before** its
  declaration, in the order `main.s` includes the files, is taken as
  sixteen bits by `ca65`, and the link fails with a range error: that use
  goes with `f:`. The header of `far_ram.s` says the same.
* **The area is all of `$7E2000-$7E7FFF`**, since 2026-10-16: 24 KB
  between bank 0's low RAM, which the console mirrors below, and
  [the grid](../optimizations/lossless/the-map/the-grid-in-wram.md) in the
  upper half of `$7E`; `$7F` is shared in time by the battle and the loads.
  Until that day most of it was a map of addresses written by hand, with
  holes between its regions and regions sized for any game, and `FARBSS`
  was a stretch of it that grew by taking over its neighbours'; that day
  every region went into the segment, as the
  [measurement](../research/2026-10-15-the-wram-and-sram-map.md) suggested,
  and the switches' table, `SWITCHES`, went into the same area. `just
  space` prints what the whole area has left.
* **A region whose size the game decides is sized by the game**: its name
  and its why stay in `far_ram.s`, which comes before the cartridge's
  `.inc`, and its `.res` goes in `far_ram_game.s`, right after it
  —`ACTORS_N`, `VARIABLES_N`, `EXP_K`…—. That is also what keeps one table
  of a game with many actors from running into the next.
* **Many keep sixteen-bit names**: the code that runs with the data bank
  at `$7E`, or points the WMADD window at them, names them by their
  `.loword`, so the label is lowercase and the name the code uses is a
  constant: `far_battle` and `FAR_BATTLE = .loword(far_battle)`. A block
  walked as a range —what `clear_variables` clears in one pass, each line
  of `save_table`— is one `.res` with offsets, or several in a row with an
  `.assert` that they are adjacent: the linker places them in the order of
  the text, and nothing else promises it. A size the `.res` needs before
  the layout that defines it exists is written as a number, with an
  `.assert` that it is exactly that (`TL_BYTES`).
* **The same holds for what is in `low_ram.s` and `zero_page.s`**: it stays
  where it is.
* **What has a reason to be at a given address is an area of the `.cfg`,
  not a number in the runtime**: the areas are `WRAM_AREAS` in
  `banks.rs`, and a segment `.res` declares what is in them. The grid's
  two halves are fixed because the converter writes them as numbers into
  what it compresses, so its own copy goes in the `.inc` and the runtime
  asserts they are the linker's; `GRID_TOP` is a segment after the grid's,
  which the grid's `.res` pushes to the end of the half. **What borrows a
  region for a while** —the battle's scene, a full screen's image, a big
  picture's palette over the ChipSet animation's drawings— **is named
  after it**, an offset with an `.assert` of how far it goes, not a second
  copy of its address.
* **The far bank is not a place**: `$7E` is where bank 0's RAM shows again,
  and the code that runs with the data bank there counts on it, so it is
  written as `$7E0000 +`, `#$7E` and a zero in `WMADDH`, and
  `far_ram_game.s` asserts `FARBSS` is in it.
* **What guards it**: ABL's `tests/tooling/test_wram_reservations.py`,
  in the-advocate-ports, looks in the runtime's source for a WRAM address
  written by hand —any long one but the far bank, or a sixteen-bit one
  named `FAR_*`—, since in the symbol table a `.loword` of a label and a
  number written by hand look the same; and it checks, for every area
  `rm2k-snes-cartridges assemble --memory-map` lists, that every long symbol in it is
  a label of its segments or falls inside one.
