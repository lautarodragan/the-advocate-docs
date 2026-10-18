# Lossless optimizations

What saves ROM, RAM, VRAM, CGRAM, ARAM or CPU time **without changing
what is seen or heard**. Where an entry has a narrow exception —a case
where something does change— it says so; a full trade goes in
[Lossy](lossy.md).

One page per part of the port:

## [The map](lossless/the-map.md)

* [The 16x16 block grid](lossless/the-map.md#the-16x16-block-grid)
* [A pointer that walks with the hero](lossless/the-map.md#a-pointer-that-walks-with-the-hero)
* [The grid, its blocks and the bytecode, in LZ2 and decompressed into WRAM](lossless/the-map.md#the-grid-its-blocks-and-the-bytecode-in-lz2-and-decompressed-into-wram)
* [The map's tiles in LZ2](lossless/the-map.md#the-maps-tiles-in-lz2)
* [Only the tiles the map draws go to VRAM](lossless/the-map.md#only-the-tiles-the-map-draws-go-to-vram)
* [The VRAM layout is chosen map by map](lossless/the-map.md#the-vram-layout-is-chosen-map-by-map)
* [The CGRAM split is chosen map by map, background first](lossless/the-map.md#the-cgram-split-is-chosen-map-by-map-background-first)
* [Opaque fixed events baked into the map](lossless/the-map.md#opaque-fixed-events-baked-into-the-map)
* [A painted map keeps only what the screen shows](lossless/the-map.md#a-painted-map-keeps-only-what-the-screen-shows)
* [A panorama gets only the layer it needs](lossless/the-map.md#a-panorama-gets-only-the-layer-it-needs)
* [The pictures that stay put are drawn into the map](lossless/the-map.md#the-pictures-that-stay-put-are-drawn-into-the-map)
* [The tilemap is a window the camera fills at its edges](lossless/the-map.md#the-tilemap-is-a-window-the-camera-fills-at-its-edges)
* [The ChipSet animation is one DMA, on the tick it changes](lossless/the-map.md#the-chipset-animation-is-one-dma-on-the-tick-it-changes)
* [The panorama's tilemap is a ring the camera fills too](lossless/the-map.md#the-panoramas-tilemap-is-a-ring-the-camera-fills-too)
* [A grid of at most 256 blocks takes one byte a cell](lossless/the-map.md#a-grid-of-at-most-256-blocks-takes-one-byte-a-cell)
* [A cell's passability is worked out when the cartridge is built](lossless/the-map.md#a-cells-passability-is-worked-out-when-the-cartridge-is-built)
* [The load buffer is lent while the map is played](lossless/the-map.md#the-load-buffer-is-lent-while-the-map-is-played)
* [Coming back from a screen reloads only what it overwrote](lossless/the-map.md#coming-back-from-a-screen-reloads-only-what-it-overwrote)

## [Characters and sprites](lossless/characters-and-sprites.md)

* [Poses are uploaded on the fly](lossless/characters-and-sprites.md#poses-are-uploaded-on-the-fly)
* [A pose goes up without its empty tiles](lossless/characters-and-sprites.md#a-pose-goes-up-without-its-empty-tiles)
* [Distant still characters take turns on one slot](lossless/characters-and-sprites.md#distant-still-characters-take-turns-on-one-slot)
* [The sprite slots as a cache](lossless/characters-and-sprites.md#the-sprite-slots-as-a-cache)
* [The possible leaders share one CGRAM slot](lossless/characters-and-sprites.md#the-possible-leaders-share-one-cgram-slot)
* [The graphics only the hero shows go in the leaders' slot](lossless/characters-and-sprites.md#the-graphics-only-the-hero-shows-go-in-the-leaders-slot)
* [Finished routes give up their slot](lossless/characters-and-sprites.md#finished-routes-give-up-their-slot)
* [Battle animations: one box for the whole ROM](lossless/characters-and-sprites.md#battle-animations-one-box-for-the-whole-rom)
* [A battle animation's frame: without its empty tiles, one at a time](lossless/characters-and-sprites.md#a-battle-animations-frame-without-its-empty-tiles-one-at-a-time)
* [Still events that animate are walked only when a column changes](lossless/characters-and-sprites.md#still-events-that-animate-are-walked-only-when-a-column-changes)
* [The timer and the raindrops fill the last row's free columns](lossless/characters-and-sprites.md#the-timer-and-the-raindrops-fill-the-last-rows-free-columns)
* [A small picture: its sparse blocks in 8x8, its repeated tiles once](lossless/characters-and-sprites.md#a-small-picture-its-sparse-blocks-in-8x8-its-repeated-tiles-once)

## [Events](lossless/events.md)

* [A common event's shared body, called with `OP_CALL`](lossless/events.md#a-common-events-shared-body-called-with-op_call)
* [`CallEvent` pasted in place, with no interpreter stack](lossless/events.md#callevent-pasted-in-place-with-no-interpreter-stack)
* [Identical pages are stored once](lossless/events.md#identical-pages-are-stored-once)
* [Copies of an event share a row](lossless/events.md#copies-of-an-event-share-a-row)
* [The pure parallel common events run only when what they read changes](lossless/events.md#the-pure-parallel-common-events-run-only-when-what-they-read-changes)
* [The refresh runs at three moments](lossless/events.md#the-refresh-runs-at-three-moments)
* [A refresh only when something changes, and only for what changed](lossless/events.md#a-refresh-only-when-something-changes-and-only-for-what-changed)
* [One dispatch table for the opcodes](lossless/events.md#one-dispatch-table-for-the-opcodes)
* [Choosing the active pages reads one byte of the pages with nothing to choose](lossless/events.md#choosing-the-active-pages-reads-one-byte-of-the-pages-with-nothing-to-choose)
* [The refresh leaves its answers for the frames that follow](lossless/events.md#the-refresh-leaves-its-answers-for-the-frames-that-follow)
* [`OP_CELL_TEST`: four commands in one opcode](lossless/events.md#op_cell_test-four-commands-in-one-opcode)
* [A parallel that stops at its cell test runs without its interpreter](lossless/events.md#a-parallel-that-stops-at-its-cell-test-runs-without-its-interpreter)
* [The commands the parallel pages run every frame take the short way](lossless/events.md#the-commands-the-parallel-pages-run-every-frame-take-the-short-way)

## [Text and windows](lossless/text-and-windows.md)

* [A repeated box is stored once](lossless/text-and-windows.md#a-repeated-box-is-stored-once)
* [DTE in the messages](lossless/text-and-windows.md#dte-in-the-messages)
* [Glyphs pre-rendered in the cartridge](lossless/text-and-windows.md#glyphs-pre-rendered-in-the-cartridge)
* [The message window's text area from one line](lossless/text-and-windows.md#the-message-windows-text-area-from-one-line)
* [One tilemap, two planes, placed by HDMA](lossless/text-and-windows.md#one-tilemap-two-planes-placed-by-hdma)
* [A face borrows the window's text cells and, by HDMA, palette 1](lossless/text-and-windows.md#a-face-borrows-the-windows-text-cells-and-by-hdma-palette-1)
* [The screens' text takes tiles only where it has ink](lossless/text-and-windows.md#the-screens-text-takes-tiles-only-where-it-has-ink)
* [The screens' text is typeset while the frame waits for the vblank](lossless/text-and-windows.md#the-screens-text-is-typeset-while-the-frame-waits-for-the-vblank)
* [The window's arrow and the shop's windows go up only when they change](lossless/text-and-windows.md#the-windows-arrow-and-the-shops-windows-go-up-only-when-they-change)

## [Battle](lossless/battle.md)

* [The scene and the windows are one layer](lossless/battle.md#the-scene-and-the-windows-are-one-layer)
* [Each distinct troop page once](lossless/battle.md#each-distinct-troop-page-once)
* [The battlers and the light backdrops in LZ2](lossless/battle.md#the-battlers-and-the-light-backdrops-in-lz2)
* [Encounters and experience, precomputed](lossless/battle.md#encounters-and-experience-precomputed)
* [The battlers in the grid's half of `$7E`](lossless/battle.md#the-battlers-in-the-grids-half-of-7e)
* [A dead enemy redraws only its cells](lossless/battle.md#a-dead-enemy-redraws-only-its-cells)

## [Audio](lossless/audio.md)

* [The stream is per note, with repeats](lossless/audio.md#the-stream-is-per-note-with-repeats)
* [A note names its sound in one byte](lossless/audio.md#a-note-names-its-sound-in-one-byte)
* [A single sample bank, uploaded in packages per map](lossless/audio.md#a-single-sample-bank-uploaded-in-packages-per-map)
* [Effects share places in the directory](lossless/audio.md#effects-share-places-in-the-directory)
* [Identical samples share their entry](lossless/audio.md#identical-samples-share-their-entry)
* [A package's effects lie in the `.msu` in the order they go up](lossless/audio.md#a-packages-effects-lie-in-the-msu-in-the-order-they-go-up)
* [With MSU-1, the themes with a track upload no samples](lossless/audio.md#with-msu-1-the-themes-with-a-track-upload-no-samples)
* [Boot is sized for the boot package](lossless/audio.md#boot-is-sized-for-the-boot-package)

## [CPU time in the frame](lossless/cpu-time-in-the-frame.md)

* [The hottest NPC code in bank 1, unrolled](lossless/cpu-time-in-the-frame.md#the-hottest-npc-code-in-bank-1-unrolled)
* [The tint's tables are copied, not rebuilt](lossless/cpu-time-in-the-frame.md#the-tints-tables-are-copied-not-rebuilt)
* [A character flash divides once per channel](lossless/cpu-time-in-the-frame.md#a-character-flash-divides-once-per-channel)
* [An NPC off screen is discarded before its row is built](lossless/cpu-time-in-the-frame.md#an-npc-off-screen-is-discarded-before-its-row-is-built)
* [The raindrops go straight to OAM](lossless/cpu-time-in-the-frame.md#the-raindrops-go-straight-to-oam)
* [A DSP write the register already has is skipped](lossless/cpu-time-in-the-frame.md#a-dsp-write-the-register-already-has-is-skipped)
* [The code runs from the FastROM mirror](lossless/cpu-time-in-the-frame.md#the-code-runs-from-the-fastrom-mirror)
* [LZ2 decompressed with block moves](lossless/cpu-time-in-the-frame.md#lz2-decompressed-with-block-moves)
* [One vblank budget, counted in poses](lossless/cpu-time-in-the-frame.md#one-vblank-budget-counted-in-poses)
* [The tint is a table per channel](lossless/cpu-time-in-the-frame.md#the-tint-is-a-table-per-channel)
* [The screen flash walks its table](lossless/cpu-time-in-the-frame.md#the-screen-flash-walks-its-table)
* [A moving event's row and its slot are remembered](lossless/cpu-time-in-the-frame.md#a-moving-events-row-and-its-slot-are-remembered)
* [The events' passability is looked at only when some row imposes it](lossless/cpu-time-in-the-frame.md#the-events-passability-is-looked-at-only-when-some-row-imposes-it)
* [The NPC table is copied to RAM when the map loads](lossless/cpu-time-in-the-frame.md#the-npc-table-is-copied-to-ram-when-the-map-loads)
* [A picture's palette, only when its tone really changed](lossless/cpu-time-in-the-frame.md#a-pictures-palette-only-when-its-tone-really-changed)
* [The timer counts seconds](lossless/cpu-time-in-the-frame.md#the-timer-counts-seconds)

## [The cartridge](lossless/the-cartridge.md)

* [Identical blocks are stored once](lossless/the-cartridge.md#identical-blocks-are-stored-once)
* [Tables of pointers, stored once by their text](lossless/the-cartridge.md#tables-of-pointers-stored-once-by-their-text)
* [The fast half first](lossless/the-cartridge.md#the-fast-half-first)
* [What does not fit, in the `.msu`](lossless/the-cartridge.md#what-does-not-fit-in-the-msu)
* [A save slot per SRAM bank](lossless/the-cartridge.md#a-save-slot-per-sram-bank)
* [The far RAM tables are as big as the game asks](lossless/the-cartridge.md#the-far-ram-tables-are-as-big-as-the-game-asks)
