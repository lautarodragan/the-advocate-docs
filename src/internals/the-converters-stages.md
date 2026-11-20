# The converter's stages, in order

The [optimizations](../optimizations/README.md) list what the port does to
make a game fit, one entry each. This page says **in which order the
converter runs them**, which budget each one spends or frees, and which
ones compete, so that whoever adds one knows where it goes.

The order matters for two reasons, and they are different:

- **A lossy step is not commutative.** It throws something away, and what
  runs after it cannot get it back: if the RAM's trim leaves an event out
  before Wally would have painted it into the map, the painting never sees
  it.
- **A lossless step gives the same picture in any order, but it spends a
  budget**, and the budgets are shared. Painting a map frees sprite slots
  and costs VRAM layouts with fewer sprite tiles; baking an event frees an
  object and costs the background colours. Whoever comes first takes the
  room.

Nobody declares the order: it is how `convert::run`, in `convert.rs` of
rm2k-snes-convert, asks its stages, and how each stage orders its own
attempts. What follows is read from there. The binaries and what each
leaves behind are in [the pipeline](pipeline.md); the list of what a
cartridge runs with less than RM2k, `degradations.json`, too. The names in
`code` below are the `kind`s that list counts by, as the converter writes
them.

## The budgets

| Budget | Where it is | Who spends it |
|---|---|---|
| **Event rows** | `EV_MAX` rows of the event tables, in RAM (`record.rs` in rm2k-snes-mapdata; see [the WRAM map](wram.md)) | every event the map keeps, and the copies of the common events |
| **The object table** | `NPC_MAX` drawn events (`sprites/npc.rs` in Wally) | every event drawn as an object |
| **Background tiles and blocks** | the layout's `bg` tiles, and the map's blocks in the WRAM grid ([Maps](maps.md)) | the map's layers, and everything baked, decorated, posed or painted into them |
| **Sprite tiles and slots** | the layout's `sprites` tiles ([the VRAM split](vram.md)) | characters, the timer, the raindrops, a picture, the battle animation box |
| **CGRAM** | the lower half for the background and the layer above, the upper for the objects ([the CGRAM split](cgram.md)) | the window first, then the background, then the layer above; eight object palettes |
| **ROM banks** | `just space` | everything the emission writes; what does not fit, to the `.msu` when the game requires it |
| **ARAM** | each map's audio package | the music and the effects, decided last |
| **Frame time** | the vblank and the CPU | objects placed every frame; the cells a posed event or a painted map rewrites; a seek into the `.msu` |

## The order

### 1. The RAM's trim

`Game::trim`, in `game.rs`, with `trim.rs`: **before anything reads a
map**, a map with more events than `EV_MAX` rows loses some, and from then
on every stage numbers its events without them (they travel as
`dropped_events`). It frees event rows; it is the first lossy step.

- **Lossless first**: the twins —events that do the same thing— share one
  row.
- **Then lossy**, by `Loss`, least first: what does nothing, a still wall
  (closed in the passability), a piece Wally will paint (`Loss::Painted`),
  only looks, walls, pages, exits —one exit left out keeps a row for a box
  that says so—, and what somebody names.
- **Reports** `event-left-out`, with what each one loses.

The trim cannot wait for Wally, which needs its numbering, so it is given
Wally's rule for a painted piece (`painted_piece`) and the maps the
project names; whether Wally then can paint each map, it does not know
([draft](https://github.com/lautarodragan/the-advocate-work/blob/main/tasks/the-trim-guesses-which-maps-go-painted.md)).

### 2. The scan, and what it decides for every map

The event compiler's first call (`compiler.scan`) numbers the whole game's
variables and switches: a variable past `VAR_SLOT_MAX` has no slot in far
RAM, and the pages that name one are left out (a `WARNING` line, and the
compiler's own losses). Then the converter decides, for every map at once:

- **the automatic common events with pictures, copied into each map** as
  one more event (`common_pages.rs`) —only on a map the trim left a free
  row in: the copies come after the trim and spend what it left;
- **which maps carry the timer's digits, the raindrops and the sandstorm's
  grains** (`scan.rs`): each costs those maps sprite tiles and a palette's
  room, decided before Wally sees the map;
- **where the hero can arrive** (`reach.rs`), which the layer above, the
  posed events and the painted pieces ask for.

### 3. The window's colours

`screens::window_colors`, before any map: the message window's colours are
reserved at the bottom of CGRAM for every map's background, so every
background starts with fewer. **Lossy** when a System does not fit:
`window-text-colour-merged`, `window-frame-fused`.

### 4. The pictures drawn into their map

`map_pictures::decide`, in the converter: which `ShowPicture`s stay put
and are drawn into the map instead of shown ([lossless](../optimizations/lossless/the-map.md#the-pictures-that-stay-put-are-drawn-into-the-map)).
A translucent big one the rule leaves as a picture is shown over black:
`translucent-picture-hides-the-map`.

### 5. Each map: Wally

`maps::prepare` builds each map's request and `maps::run_all` asks `wally
map` for all of them, in parallel. Inside, `map::convert` in Wally does
this, in this order, in `convert_with`:

1. **Which opaque tile events can be baked** into the layer above
   (`baking.rs`), and, if the project asks, which **decorative** events
   could go into the background (`decor.rs`).
2. **The still pieces painted**, if `[graphics] painted_events` names the
   map and the map can be one picture (`painted_pieces`, `pieces.rs`): they
   replace the decorative ones, and the pieces the trim left out are
   painted too, read from the whole map. Frees event rows, object table
   entries, slots and frame time; costs the map a [painted
   layout](vram.md#a-painted-map-keeps-only-what-the-screen-shows) —256
   sprite tiles— and every cell of itself in the ROM.
3. **The layers drawn and converted** —Adeleine decides which colours are
   lost and the palettes, Voodoo cuts the tiles—, with **the tiles above
   the hero nobody can cover lowered** into the background only if that
   costs nothing (`lowering_costs`): no other panorama way, no colour more
   merged, no cell more fused.
4. **The baking**, kept only if the CGRAM split does not move
   (`bake_unless_split_moves`).
5. **The decoration**: all of it if the split holds (`same_split`), or
   dithered (`translucent-dithered`), or CharSet by CharSet, the most
   numerous first, keeping what fits.
6. **The rows a route dresses** (events that draw nothing until a route
   gives them a graphic), each a hidden row.
7. **The object table**: past `NPC_MAX`, some are not drawn
   (`events-not-drawn`), and still run.
8. **The posed events** —still events that change pose— into the
   background's cells (`posed_events`, `posed.rs`), if they cost nothing
   but background tiles it has room for. Frees objects and frame time;
   costs background tiles, and a cell rewrite when the pose changes.
9. **The characters**: the eight object palettes split among them
   (`palette-lent`, `timer-colors-lent`, `rain-colors-lent`; seven
   palettes on a map with the battle animation box) and the sprite bank.

That is one conversion. `convert_fitting` then checks it **against the
VRAM layouts**, and when it fits none, tries again giving up something,
in this order —each step loses more than the one before, and it stops at
the first that fits—:

1. as it is;
2. if it shows pictures: fewer still events animating
   (`still-animation-dropped`), and if that is not enough, without the
   pictures (`pictures-not-drawn`);
3. the still events taking turns on a slot ([lossless](../optimizations/lossless/characters-and-sprites.md#distant-still-characters-take-turns-on-one-slot));
4. a slot cache ([lossless](../optimizations/lossless/characters-and-sprites.md#the-sprite-slots-as-a-cache)),
   where events may wait for a slot (`events-wait-for-a-slot`);
5. fewer still events animating (`still-animation-dropped`);
6. without the battle animation box (`battle-animations-not-drawn`);
7. the background's most alike cells fused, Adeleine's `group_tiles`
   (`background-tiles-fused`).

Around that ladder, two more comparisons of the whole map: **the common
events' copied pictures** go in only if the map loses nothing more with
them (`pictures-not-drawn` names theirs), and **the dressed rows** only if
they cost nothing else (`dressed-not-drawn`). The rest of what Wally
reports —`rain-not-drawn`, `sand-grains-not-drawn`— comes from the extras
at the end of the sprite bank.

### 6. The grid in WRAM

After Wally, because it needs the block count: a map whose grid fits WRAM
neither at two bytes a cell nor at one is left out (`map-left-out`), and
teleports to it become a box. See `cell_bytes` in rm2k-snes-mapdata and
[Maps](maps.md#the-grid-lives-in-wram-and-the-cartridge-carries-it-compressed).

### 7. The big pictures, and the pages

The big pictures are numbered for the whole cartridge (`pictures.rs`): the
maps' first, then the common events', which lose theirs past the table's
byte. Then `rm2k-snes-battle` answers the encounters and the experience,
and the event compiler compiles every page, with its own losses per
command (`rm2k-snes-events culprits`); what Wally painted, drew into the
map or shows elsewhere is forgotten from that list. The second round
checks what needs every map: the route slots (`route-slots-short`), the
loops (`map-does-not-wrap`), a flash on a shared palette
(`flash-shared-palette`).

### 8. The screens and the battle

The screens, all at once; the title at 8 bits, or at 4 if it does not fit,
and none if not even its menu fits in VRAM (`title-screen-not-drawn`); the
Game Over; the screen that asks for the MSU-1. Then the battle: the
animations the bytecode shows, the backdrops (`backdrop-only-in-msu`), the
troops (`troop-members-left-out`) and the model (`names-cut`,
`text-pool-short`).

### 9. The emission

`rm2k-snes-mapdata` writes the `.inc`. It decides how the cartridge
carries what the others answered, losslessly: **the layout**, the first of
the [VRAM split](vram.md)'s that fits (`vram_layout`); the blocks stored
once; the LZ2 streams and whether each fits where the console decompresses
it. And two lossy things whose caps are its own: a ChipSet animation that
does not fit in WRAM stands still in part (`chipset-animation-cut`), and an
inn's gold window with no room (`inn-no-gold-window`).

### 10. The patches, then the music

`rm2k-snes-patches` writes what Adeleine decided —which colour went into
which— as the game's patches: decisions, not degradations. **The music
runs last**, `rm2k-snes-audio cartridge`, because what a map's package
carries depends on everything above: it spends ARAM, and loses
`songs-uploaded-on-play`, `sound-effects-left-out` or
`package-waits-for-playbgm`. It also writes the `.msu`, with the battle
backdrops and the effects. Only then is `degradations.json` written.

### 11. The packing

`rm2k-snes-assemble`, after the converter (`banks.rs` in
rm2k-snes-cartridge): **first-fit decreasing**, the fast 4 MB before the
slow ones. In a game with `[rom] msu = "required"`, what does not fit goes
behind the `.msu` (`pack_with_msu`): the fewest pieces that make the rest
fit, the largest first. Each one costs a seek when it loads. See [the
`.msu`](the-msu-file.md#what-does-not-fit-in-the-rom).

## The pairs known to compete

| First | Then | What showed it |
|---|---|---|
| The RAM's trim (1) | The painted pieces (5.2) | Desert Nightmare's towns: the trim left out the roofs the painting could have kept, and they showed white on maps 4, 14, 61 and 66. Fixed by telling the trim which maps go painted (`Loss::Painted`) and having Wally paint the left-out pieces from the whole map ([task](https://github.com/lautarodragan/the-advocate-work/blob/main/tasks/desert-nightmare-roofs-left-out-show-white.md)) |
| The painted pieces (5.2) | Sprite slots (the ladder) | The painted layout keeps 256 sprite tiles: Desert Nightmare's maps 18, 33 and 71 went on to wait for slots, and 42 one-screen interiors lost the message box's two pictures. They are not painted: the list of maps is by hand ([task](https://github.com/lautarodragan/the-advocate-work/blob/main/tasks/painted-events-chooses-its-maps.md)) |
| The painted pieces (5.2) | ROM banks (11) | Painting every map Desert Nightmare benefits from does not fit beside the rest: it [requires the MSU-1](../ports/desert-nightmare/decisions/the-msu-1-is-required-for-now.md) |
| The trim (1) | The common events' copies (2) | A map the trim filled has no row for the copy, and its common event's pictures run blind there (`common_pages.rs`) |
| The window's colours (3) | Every background | The window takes its entries from every map first: [lossy](../optimizations/lossy/colours.md#the-message-window-takes-its-colours-from-every-background) |
| The tiles above lowered (5.3) | Background colours and tiles | Lowered unconditionally, 36 of Vampires Dawn's maps merged more colours, one of them 111 (2026-11-02): now only if it costs nothing |
| The posed events (5.8) | Background colours | Zounds' desert, map 34, came out a shade off with the same number of colours merged (2026-11-02), and its ship capsule, map 17: now only if the background merges the very same colours |
| The posed events drawn past their cells (5.8) | The panorama's colours | Letting them move the CGRAM split, the Hover-Tram's panorama went from 46 colours kept to 32: the split may move only if the panorama keeps what it kept (`only_the_split_moved`) |
| Still events animating (the ladder) | Slots in a cache | Vampires Dawn's maps 236 and 240 went from no wait to waiting when their stills animated: fewer animate first |

## Where a new optimisation goes

**A lossless one goes as early as the budget it frees is decided**, and
must say what it spends: if it spends a budget an earlier step already
divided —the CGRAM split, the layouts— it has to be tried against that
split and kept only if the split does not move, as baking, decorating,
lowering and posing are. **A lossy one goes as late as it can**: after
every lossless way of fitting, and after every lossy one that loses less,
because what it throws away nobody after it can see. And **a step that
runs before another one that could have saved what it drops has to be
told about it**, as the trim is told which maps go painted: the trim
cannot wait for Wally, which needs its numbering, so it is given Wally's
rule instead. If the order is wrong somewhere, the symptom is a
degradation that a later step would have avoided; compare the map's
`degradations.json` with the step turned off.
