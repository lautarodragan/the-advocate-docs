# What ExFont icons do not model

## What

In RPG_RT a `$` and a letter, `$A` to `$Z` and `$a` to `$z`, is one of the
52 icons of its ExFont, twelve pixels square, in a name, a description or
a message alike —easyrpg's `Utils::ExFontNext`—; any other `$` is a `$`,
and in a message `\$` is the gold window's escape. The icons are drawn in
the text's colour, like a letter. The console draws them since 2026-11-02,
from the game's own ExFont:

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
* **Every code, in a game without an ExFont**: one with no `RPG_RT.exe`
  and no sheet of its own, like the test game; and in one whose project
  file says `[text] exfont = false`, for a cartridge with no room for
  their glyphs —each half of an icon a box draws is 1,248 bytes of the
  window's tables—. None says it yet; ABL's release, which draws two in a
  sign's glitched text and has no bank left, is the closest —its gaps,
  `just space` in its port—.

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

**The icons are the game's**, read from where RPG_RT keeps them, its
executable: EasyRPG's free ExFont is part of easyrpg-player, under the
GPLv3, and the-advocate carries nothing of a third party without a licence
it can go in.

**An icon is two glyphs**, its left and right halves, six columns each:
the composer takes glyphs at most eight columns wide —with the shadow, two
cells and no more from any column—, and the halves, drawn one after the
other, each with its shadow, are the icon drawn whole, the right half's ink
over the left half's shadow where they meet, as a glyph's own ink covers its
shadow. They keep RM2k's six pixels each —an icon is twelve wide, as in
RM2k— and `letter_spacing` does not widen them.

## How it applies

**Where the icons come from**: `rm2k_render::exfont`, easyrpg's order —a
sheet of the game's own, `Font/ExFont` or `ExFont`, PNG or BMP, then
`RPG_RT.exe`'s `BITMAP` resource `EXFONT`—, or the file the project file's
`[text] exfont` names in the game's directory, for a game that runs with
another executable. The converter finds it and hands its path to the
stages, `exfont`, next to the font's name; `Font::with_exfont` adds the
104 halves to the font.

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
`rm2k-snes-events` and `rm2k-snes-screens`. Task:
[`rm2k-s-exfont-icons-in-names-and-texts`](https://github.com/lautarodragan/the-advocate-work/blob/main/tasks/rm2k-s-exfont-icons-in-names-and-texts.md).
