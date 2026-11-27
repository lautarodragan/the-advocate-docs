# The pause menu and its options

The pause menu is RM2k's `Scene_Menu` at 256x224: the command column on the
left, the party on the right and the gold at the bottom, over black, in the
same tile bank and palette as the save screen. B opens it when the hero is
standing still, and the game clock does not run while it is up. The code is
`runtime/menu.s` in the-advocate, and the geometry and the blocks come from
`rm2k-snes-screens`.

## The options

The column is a list of **rows**, and each row shows a **command**: the
runtime builds the list each time the menu opens —`menu_rows_build`, in
`runtime/custom_menu.s`—, so an option can come and go without the others
knowing. In order:

| Row | Command | What it opens |
|---|---|---|
| RM2k's first three | Items, Skills, Equipment | the item screen, and the skills and equipment screens after choosing whom on the menu itself; grey with the party empty |
| one per custom menu | the game's custom menus | a full-screen menu the game describes as data, while its condition holds —the console in A Blurred Line; see [the custom menu](../research/2026-10-04-the-custom-menu.md)— |
| RM2k's last two | Save, End Game | the slot screen, grey while `ChangeSaveAccess` forbids it; the End Game screen |
| the last one | Debug | the debug screen, only with `[debug] debug_screen` and the MSU-1 (below) |

RM2k's five are commands 0 to 4, the custom menus 5 and on, and the debug
screen the one after the last custom menu. The labels are drawn by
`rm2k-snes-screens` into the menu's bank, two blocks each —the normal one
and the grey one—, by command; the row they go on is the runtime's. Every
option that opens a full screen —the custom menus, the skills screen, the
debug screen— takes the whole VRAM, and the map comes back as from a big
picture when the menu closes: `CM_DIRTY`, and `big_back_far`.

## The debug screen

RPG_RT's F9 —`Scene_Debug`, only in a test play—, which the console has no
key for: the switches and the variables of the database, turned on and off
and changed by hand. It is `runtime/debug_screen.s` and
`rm2k-snes/screens/src/debug_screen.rs`, and it is what RPG_RT 2000 shows,
measured under Wine:

* the ranges on the left, ten rows of `S[0001-0010]`, and the list on the
  right, ten rows of `0001: Name` with `[ON]` or `[OFF]`, or a variable's
  value right-aligned;
* in the ranges, up and down go round the page's ten, left and right flip
  pages of ten ranges, round, and after the switches' pages come the
  variables' —`V[0001-0010]`—; A goes into the list and B leaves;
* in the list, up and down go round its rows, A turns a switch on or off,
  left and right change a variable by one and L and R by ten, and B goes
  back to the ranges.

**A change takes effect when the map comes back**, as in RPG_RT: a switch
or a variable that changes asks for the pages' refresh, and the pure
parallel common events run again when the menu closes. Where it differs
from RPG_RT is in [What the debug screen does not
model](../divergences/what-the-debug-screen-does-not-model.md).

**It is there only with the MSU-1**, since 2026-11-16: its font, its
names and its windows travel only in the `.msu` —taro tests with it, and
the screen is not needed to enjoy the game—, so without the MSU-1, or with
another build's `.msu`, the pause menu does not offer it and End Game is
its last row. What it reads from there and when, in [the `.msu`
file](the-msu-file.md#the-debug-screen).

**It runs in the console's 512-wide mode**, mode 5, since 2026-10-20
—taro's: at 256 wide a third of A Blurred Line's switch names were cut—.
Its pixels are half as wide on screen, so a line holds twice as many; the
windows are drawn at 256 wide like every other screen's and go up with
each pixel twice, so they look as they did.

**It writes with a font of its own**, since 2026-11-01, in every game and
whatever the game's: `hires-wide`, in `rm2k-snes/text/fonts/` —its
README says how it is drawn—. Until then it wrote with the game's font,
whose pixels at half width left every vertical stroke one pixel of the
512, and on a CRT taro found it "practically unreadable"; this one has
them two pixels wide, and is about 1.4 times as wide as the game's at 256.
It is not a knob: no game can use it as its font —a letter moves the line
up to ten pixels, more than a cell of the other screens— and there is no
second one to choose. The cartridge carries its tables apart, which only
the 512-wide mode reads, and of its Latin-1 the letters the database's
names use, so `Händler` is written as such. See
[the task](https://github.com/lautarodragan/the-advocate-work/blob/main/tasks/a-wider-font-for-the-512-wide-debug-screen.md).

In mode 5 a tilemap word is a cell 16 pixels wide, two tiles side by side:

* **the text is BG1, at 4bpp**, with the text layer in its wide mode
  —`tl_hires` in `runtime/text_layer.s`—: a column of the layer is four
  tiles, a *quad*, where elsewhere it is a pair, and its inks are the
  items' white and grey, moved to palettes 2 and 3; the screen's upper
  half and its lower half take their quads from two pools, each its own
  1024 tiles of the VRAM, which an HDMA switches at line 112 —see
  [VRAM](vram.md#the-debug-screen-is-512-wide)—;
* **the windows are BG2, at 2bpp**: eight palettes of three colours in
  the first 32 entries of the colour memory, which Adeleine's
  `reduce_colors` fills from the System —one with more colours, or more
  than three in a cell, loses some; A Blurred Line's has three—;
* **the main screen and the subscreen show both layers**, which is what
  mode 5 needs to show every pixel of the 512; no objects.

When B leaves, mode 3 and no subscreen again, which is what the pause menu
draws in. The VRAM, in [VRAM](vram.md#the-debug-screen-is-512-wide).

The converter measures the screen with that width: the ranges' window is
as wide as its widest label, and a name is cut only if it does not fit in
what is left —none of A Blurred Line's—. The names go in a block of their
own in a far bank, by reference; the code is in `FAR3`. A cartridge that
does not ask for the screen carries none of it: the code is inside an `.if
DEBUG_SCREEN`, and the converter writes only `DEBUG_SCREEN = 0`.

### The state's switches

**After the variables' pages comes one that is not RPG_RT's**, since
2026-11-17: three switches —`Tint`, `Timer` and `Weather`—,
each `[ON]` while the game has its way and `[OFF]` once A forced it off.
Left from the first page goes round to it. They are for bisecting lag and
other bugs in a game as it is played, on its own save: the state the game
leaves on across maps, which a test's jump clears and a played game keeps.
taro asked for them after the opening's tint in A Blurred Line —saturation
40, kept through every door— turned out to be most of the frames each stone
lost in the Lazer Teeth HQ, and the timer on screen three more —see [the
task](https://github.com/lautarodragan/the-advocate-work/blob/main/tasks/debug-switches-for-the-global-state.md)—.

* **`Tint`** forced off leaves the screen neutral, so the palettes and the
  flashes take their untinted paths.
* **`Timer`** forced off hides its digits; the count runs on, and still
  fires the pages that ask for it.
* **`Weather`** forced off is no weather: no raindrops, no sand grains and
  no sand overlay.

**Forcing one off keeps what the game wants** and gives it back when the
switch goes on again, so turning a switch off and on changes nothing the
game sees: what the game asks for meanwhile —a `TintScreen`, a timer that
starts, a `WeatherEffects`— is kept, not shown, and a save made meanwhile
saves the game's tint and timer, not the forced ones. A reset turns them
all on again. The map takes a change when the pause menu closes, as it
takes a switch's.

**They cost nothing per frame**: the forced state is the game's own off
state, so the main loop asks nothing; the three commands look at the
switches when a page runs them, a `jsl` each in bank 0. It is
`runtime/debug_switches.s` in the-advocate, which says why the screen's
flash and shake, the pictures, the panorama, the counter and the parallel
common events are not among them. It is part of the screen, so it comes
and goes with `[debug] debug_screen`, and needs the MSU-1 like the rest of
it.

### The PPU's counts

**Under the three switches go two counts**, since 2026-11-17, and with them
the page is `D[0001-0005]`: `Range over` and `Time over`, the frames on
which the console dropped objects on some line —more than 32 objects on it,
or more than 34 slivers of 8 pixels—, since the map was entered or since A
on the row zeroed it. Every character is a 32x32 object, four slivers, so
it is time over that comes first: nine characters on a line. taro found the
first case playing, the Hover-Tram's crowd in A Blurred Line —see [the
task](https://github.com/lautarodragan/the-advocate-work/blob/main/tasks/maps-with-more-objects-on-a-line-than-the-ppu-draws.md)—.

The NMI reads `STAT77` once a frame: the PPU clears its two flags at the
end of the vblank and sets them while it draws, so at the start of the next
one they hold the whole frame. It costs a read, a mask and a branch a frame
in bank 0, and a `jsl` to bank 1 only on a frame that went over; the counts
are two words of far RAM, which the tests read, zeroed on entering a map
—not at the boot: the title screen draws with an OAM nobody uploaded, which
never shows but is in range, and counted every frame—. On the console a
forced blank keeps the last frame's flags, so a load right after a frame
that went over counts its black frames too; bsnes and snes9x clear them
anyway.

The counts are for what a played game meets, a scene that brings people in.
Each map as it is entered is counted without the ROM by the ports'
`tests/objects_per_line.py`, which enters every map of every game.

**A row of the list is the screen's whole row** for the text layer —the
range's label beside it is the same record—, so a step that changes the
list would compose the ranges again. The layer keeps a record's head when
it did not change and the rest starts further right —`tl_split`—, so
within a page the ranges are never composed again, and a step asks the
pool only for the list's columns.

## The `[debug]` knobs

The project file's `[debug]` table holds what a port turns on while it is
being debugged and off for a clean delivery. Every key is required, like
the rest of the file:

* **`debug_map`**: a map of the port's own, with a door to each key point of
  the game, where "Begin the Story" starts —see A Blurred Line's project
  file—. It has to be the only new map of the manual patches.
* **`enemy_hp`**: the battle writes the enemies' HP, which RM2k never shows;
  the converter's `--enemy-hp`.
* **`debug_screen`**: the pause menu's last option opens the debug screen,
  above, when the MSU-1 is there —its state's switches included—; the
  converter's `--debug-screen`. Off,
  the ROM is the same byte for byte as one built before the screen existed,
  and the `.msu` carries nothing of it.
* **`phasing`**: Select turns on and off the hero walking through walls and
  events, which RPG_RT's test play does while Ctrl is held, and the hero
  blinks while it is on; the converter's `--phasing`, and
  `runtime/phasing.s` in the-advocate. Off, the ROM is the same byte for
  byte as one built before it existed. It is its own key and not the debug
  screen's because every port has that one on, and this one changes how
  the hero walks: see [Phasing is a toggle on
  Select](../divergences/phasing-is-a-toggle-on-select.md).

`advocate build` reads the four and passes the flags to every cartridge
of the project.
