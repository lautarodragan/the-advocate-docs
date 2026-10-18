# Lossless: the cartridge

## Identical blocks are stored once

Every data block the converter emits —tiles, grids, block tables,
bytecode, and the rest— has an identity of its own, `<class>_<n>`, and
the map stores which one it points to: two maps with the same bytes in a
class share the block.

- **Saves**: ROM, wherever maps repeat data.
- **Where**: `Materials::store` in `rm2k-snes/mapdata/src/materials.rs`.
- **More**: [Graphics and sound, from RM2k to the SNES](../../research/graphics-and-sound-from-rm2k-to-the-snes.md).

## Tables of pointers, stored once by their text

A table whose bytes are pointers the linker sets cannot be compared by
its bytes, so it is shared by its text, which names the same labels when
it points to the same things. The per-map leader table and the per-map
table of the poses of each character that changes pose go this way: most
maps bring one another map already brought.

- **Saves**: ROM.
- **Where**: `Materials::store_text` and `Materials::store_symbolic` in
  `rm2k-snes/mapdata/src/materials.rs`; `actor_table_asm` and
  `npc_poses_asm` in `record.rs`.
- **More**: [Fitting the whole game in 8 MB](../../ports/abl/research/2026-10-14-fitting-the-whole-game-in-8-mb.md),
  the research that measured them.

## The fast half first

Only the first 4 MB of an ExHiROM cartridge are FastROM, so the packer
fills those first.

- **Saves**: CPU on reads from ROM.
- **Where**: `banks.rs` in `rm2k-snes/cartridge/src/`.
- **More**: [ExHiROM, 8 MB](../../internals/exhirom.md).

## What does not fit, in the `.msu`

A game whose ROM does not reach declares `[rom] msu = "required"`, and the
data read when a map or a battle loads —LZ2 streams, panorama and picture
tiles, samples— travels in the `.msu`. Nothing is lost, but the game needs
the MSU-1.

- **Saves**: ROM.
- **Where**: the `.msu` in front of a `.block`, marked by each emitter;
  `lz2_msu` and `msu_dma0` in `runtime/decompress.s`.
- **More**: [The `.msu` file](../../internals/the-msu-file.md#what-does-not-fit-in-the-rom),
  [The MSU-1 is required only when the ROM does not reach](../../decisions/the-msu-1-is-required-only-when-the-rom-does-not-reach.md).

## A save slot per SRAM bank

ExHiROM shows the SRAM through an 8 KB window at `$6000` in each bank
from `$20`, and each save slot takes a whole bank: slot N is bank `$20 +
N`, so every slot has the same addresses and nothing multiplies to find
one.

- **Saves**: CPU and code when saving and loading, and a slot that
  crosses no bank.
- **Costs**: 32 KB of SRAM for four slots, a whole window each whatever a
  saved game weighs; any cartridge and flash cart has them.
- **Where**: `SRAM_BANK`, `SRAM_STRIDE` and `slot_bank` in
  `runtime/save_game.s`.
- **More**: the comment on `SRAM_SLOTS` there.

## Only the items the game can give are carried

The cartridge carries the name, description and tables of the items that
can reach the inventory with its maps: the ones a `ChangeItems` gives, a
shop sells, an enemy drops or an actor starts equipped with. Each has a
place `k` in those tables, and `item_k` turns an RM2k number into it, or
into "not carried".

- **Saves**: ROM —the texts of the database's items no page gives—.
- **Where**: `possible_items` and `drop_items` in
  `rm2k-snes/convert/src/scan.rs`; `items_and_battle` in
  `rm2k-snes/mapdata/src/emit.rs`; `item_k` and `items_build_list` in
  `runtime/items.s`.
- **More**: the doc comment on `possible_items`, and the header of
  `runtime/items.s`.

## The far RAM tables are as big as the game asks

The tables of far RAM whose size depends on the game —the variables, the
items, the actors' equipment, HP, SP and graphic, the skills, the
experience, the parallels' state— take the size the cartridge's `.inc`
declares for that game, not one chosen for any game, and the linker
places them; the converter gives a variable a slot only if some page
names it.

- **Saves**: far RAM —what one game does not use, another table of it
  can—.
- **Where**: `runtime/far_ram_game.s`, which comes after the cartridge's
  `.inc`; the names and their whys, in `runtime/far_ram.s`.
- **More**: the header of `runtime/far_ram_game.s`.
