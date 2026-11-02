# What ExFont icons do not model

## What

In RPG_RT a `$` and a letter, `$A` to `$Z` and `$a` to `$z`, is one of the
52 icons of its ExFont, twelve pixels square, in a name, a description or
a message alike —easyrpg's `Utils::ExFontNext`—; any other `$` is a `$`,
and in a message `\$` is the gold window's escape. The icons are drawn in
the text's colour, like a letter. The console draws them since 2026-11-02,
from the sheet the project file names:

* **In the message boxes and a choice's options**, after the console's 96
  glyphs, with the Latin-1 letters the boxes use: the two together are at
  most 64 —see [Letters the font does not have](letters-the-font-does-not-have.md)—,
  and past that the least used go. An icon goes whole or not at all.
* **In the items, equipment, shop, skills and battle screens**: the icons
  of the database's items and skills —their names, their descriptions and
  what a skill says when it is used—.

**Where it cannot, the code is dropped**: nothing is drawn, not `$N`.

* **A troop page's box**: the battle draws those with the text screens'
  glyphs, and the event compiler writes them without its letters, so they
  drop their icons as they show their Latin-1 plain.
* **The actors' names**, an enemy's and the terms: the menu's party strips
  and a box's `\N[0]` write a name with no icon.
* **A custom menu's texts**, and the debug screen's names.
* **Past the 64 letters of the boxes**, and a box's icon the font does not
  draw.
* **Every code, in a game without an ExFont**: one whose project file
  names no sheet, like the test game; and one whose project file says
  `[text] exfont = false`, for a cartridge with no room for their glyphs
  —each half of an icon a box draws is 1,248 bytes of the window's
  tables—. None says it yet; ABL's release, which draws two in a sign's
  glitched text and has no bank left, is the closest —its gaps, `just
  space` in its port—.

Each box that drops an icon is a `letter-plain` loss in the build's
`degradations.json`, "the icon $X as nothing", next to the letters it
shows plain.

Not modelled either: Maniacs' `$[n]` and `$[x,y]`, which are not RM2k's,
and EasyRPG's coloured ExFont —an icon is a mask here, painted in the
text's colour, as RPG_RT paints it—.

## Why

taro asked for it on 2026-11-02, playing Zounds, whose items and skills
start with one —`$NUltra Poción Roja`— and the console showed the two
characters. Counted then over each port's dump: Zounds about 225 codes,
Vampires Dawn 30, Frozen World 6, ABL 4 —a glitched sign, `%$@&%$dhbdfj3`,
where RPG_RT draws icons too— and The Book of Three 3.

**The icons are a sheet in the port**, not the game's executable. RPG_RT
keeps its ExFont as a bitmap resource of `RPG_RT.exe`, and the first
version read it from there; taro asked the same day that the tools not
depend on the `.exe` a game brings, and that the sheet live in the port's
repository, named by its file. So each port that draws icons commits the
sheet its game's executable carries, extracted from it once, and the
tools never open an executable. **Without a sheet there are no icons**:
the-advocate is public and carries no default —RPG Maker's stock sheet is
Enterbrain's, and EasyRPG's free ExFont is part of easyrpg-player, under
the GPLv3—. A free sheet of our own as the default is a wishlist item,
[`a-free-exfont-of-our-own-as-the-default`](https://github.com/lautarodragan/the-advocate-work/blob/main/tasks/a-free-exfont-of-our-own-as-the-default.md).

**An icon is two glyphs**, its left and right halves, six columns each:
the composer takes glyphs at most eight columns wide —with the shadow, two
cells and no more from any column—, and the halves, drawn one after the
other, each with its shadow, are the icon drawn whole, the right half's ink
over the left half's shadow where they meet, as a glyph's own ink covers its
shadow. They keep RM2k's six pixels each —an icon is twelve wide, as in
RM2k— and `letter_spacing` does not widen them.

## How it applies

**Where the icons come from**: the project file's `[text] exfont`, the
path of a sheet —13 icons by 4, each 12x12, indexed with index 0 as the
key, a PNG or a BMP— relative to the project file, like a font of the
game's own; `rm2k_render::exfont` reads it. Nothing is looked for in the
game's directory, and no executable is read. Each port keeps its sheet as
`exfont.png` next to its `advocate.toml`: the eight that are RM2k games
carry RPG Maker's stock one, the same bytes in all eight `RPG_RT.exe`;
Zounds carries the one of its `RPG_RT_a.exe` (see [its
page](../ports/zounds/README.md)). The converter hands the sheet's path to
the stages, `exfont`, next to the font's name; `Font::with_exfont` adds the
104 halves to the font. The sheet goes into the cartridges' fingerprint, as
a font's does.

**The codes become halves** in `rm2k_snes_text::exfont`, and
`plain_text_but`, which every text the screens write goes through, turns
them so; `plain` stands in nothing for a half. A message box's lines get
theirs in `escape_tail`, where `\$` is already the gold window's.

**The boxes** carry the halves as letters, in `Window`'s slots after
Latin-1's (`Window::carrying`, which keeps an icon's halves together).
**The text screens** carry, after the console's 96 and `TL_FIGURE`'s
place, the halves of the icons `texts::screen_icons` finds, a byte each from
129 —`ICON_GLYPH_FIRST` in `rm2k-snes-screens`, `Texts::add` in
`rm2k-snes-mapdata`—; the menu's bank carries the same glyphs, since a
skill's name goes by reference from the one text the battle's list and the
actor choice share. Since the screens' glyphs and the window's no longer
match, the window's advances are `msg_font_advance` and `msg_font_reach`
and the screens' `font_advance` and `font_reach`.

Tested by `test_zounds_exfont.py` in the ports, a box and the items screen
with icons, and the crates' tests of `rm2k-snes-text`, `rm2k-render`,
`rm2k-snes-events` and `rm2k-snes-screens`. Tasks:
[`rm2k-s-exfont-icons-in-names-and-texts`](https://github.com/lautarodragan/the-advocate-work/blob/main/tasks/rm2k-s-exfont-icons-in-names-and-texts.md)
and
[`the-exfont-is-an-image-in-the-port-not-the-games-exe`](https://github.com/lautarodragan/the-advocate-work/blob/main/tasks/the-exfont-is-an-image-in-the-port-not-the-games-exe.md).
