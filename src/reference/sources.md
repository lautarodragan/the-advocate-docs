# Sources

**The numbers and the semantics come from here, not from memory**: it
already happened once that three RM2k command codes were put in from
memory and were wrong.

## RPG Maker 2000

- [liblcf](https://github.com/EasyRPG/liblcf) — the LCF format, the chunk
  ids and the command codes; Ditto's schema comes from there.
- [EasyRPG Player](https://github.com/EasyRPG/Player) — the reference
  interpreter: what each command and each move command does.
- [Game-Decompilations](https://github.com/SamidyFR/Game-Decompilations) —
  a list of decompilations, for what EasyRPG does not explain.

## The console

- [fullsnes](https://problemkaputt.de/fullsnes.htm), by nocash — the
  reference for the whole hardware.
- anomie's documents —registers, timings, the SPC700—, which most of the
  other references cite.
- The [superfamicom.org wiki](https://wiki.superfamicom.org/).
- "SNES translucency FAQ", by Bregalad (2009, revised in 2015) — color math
  explained. It is a third party's and is not reproduced here: it is found
  by its title.
- [sd2snes](https://github.com/mrehkopf/sd2snes) — the FX Pak Pro's
  firmware: how it really implements the MSU-1.
- [The Final Fantasy VI disassembly](https://github.com/everything8215/ff6)
  — text as one byte per letter that the 65816 composes, which is what the
  message window does.

## The emulators

Besides running the ROM, they are the code that is read when the
documentation is not enough. Which ones are used to test by hand and what
each one implements: [Emulators](snes/emulators.md).

- [bsnes](https://github.com/bsnes-emu/bsnes),
  [snes9x](https://github.com/snes9xgit/snes9x),
  [Mesen2](https://github.com/SourMesen/Mesen2),
  [Mesen-S](https://github.com/SourMesen/Mesen-S),
  [ares](https://github.com/ares-emulator/ares) and
  [Mednafen](https://mednafen.github.io/).

## The assembler

- [cc65](https://github.com/cc65/cc65), where `ca65` and `ld65` come from.
  It is not packaged in every distribution and is compiled from its
  sources.

## The S-DSP model

- snes_spc 0.9.0, by blargg (Shay Green, 2007), whose accurate S-DSP,
  `SPC_DSP`, the music plays each song in to measure its volume ceiling.
  blargg's site no longer answers; the copy vendored in the-advocate, in
  `snes/spc-dsp/vendor/snes_spc/` under the LGPL 2.1 or later, comes from
  the first commit of a mirror,
  [yupferris/snes_spc](https://github.com/yupferris/snes_spc) `8f0ea63`:
  that crate's README says what was checked of it and what was not.
