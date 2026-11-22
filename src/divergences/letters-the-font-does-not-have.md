# Letters the font does not have

## What

RM2k's font has Latin-1 —`é`, `ñ`, `¿`, `ü`, `ß`— and RPG_RT draws
Windows-1252's few extras too, like the ellipsis `…` and the curly quotes.
The console's font draws printable ASCII and the Latin-1 its game uses: in
the message boxes since 2026-10-16, and on the text screens since
2026-11-22. What the screens stage draws when building —End Game's
question, the title's options, the menu's commands— always drew the
database's text as it is.

* **A message box draws the Latin-1 letters its game uses**, with their own
  glyphs —an accented letter is the font's letter with the accent on top—.
  The cartridge carries only the ones its boxes use, at most 64 together
  with the halves of the ExFont icons they draw —see [What ExFont icons do
  not model](what-exfont-icons-do-not-model.md)—; how many, its
  `MESSAGE_PAIR_FIRST` in the build's `mapdata.inc` says, minus 96. A game
  in English carries none, and its cartridge is the same as before there
  were any.
* **Any other letter is shown as its plain ASCII stand-in**, the one a
  reader would write without the key: `é` as `e`, `ß` as `ss`, `¿` as `?`,
  `…` as `...`, `Œ` as `OE`. That is every letter in a box past the 64, in a
  font that does not draw it —RM2k's own fixed-width ones, a game's sheet
  without Latin-1, and the four `low-variable` leaves out, `®`, `¾`, `Ø`,
  `ø`—, every Latin-1 letter in a **troop page's box** —the battle draws
  those with the text screens' glyphs, and its boxes do not carry them yet,
  so in Zounds every battle's talk is plain—, a letter the text screens
  would reach past glyph 223 —see below—, and the characters past Latin-1,
  like Sunset's `…`.
  Each command whose box shows one is listed in the build's
  `degradations.json` as a `letter-plain` loss, with the letters and what is
  drawn for each, and the converter prints one `WARNING, letter-plain` line
  with all of them.
* **The text screens draw the letters the database's texts use**: the
  menu, the items, equipment, skills and status screens, the shops, the
  battle's lists, help line and messages, the enemies' names, the slot
  screen, and a name written when a box opens —`\N[0]`, a renamed actor—:
  Zounds' hero is Unamë there too. They name the same glyphs as the boxes,
  and the letters only they write go after the boxes' in the one font. A
  screen's byte is its glyph plus 32, so they reach glyph 223 and no
  further: a letter past it is shown on them as its stand-in, and the
  converter says so —no game is near it; Zounds, the busiest, ends at
  187—. A name baked into a box —see
  [What names in messages do not model](what-names-in-messages-do-not-model.md)—
  is part of the box's text, and keeps its `ë` as it always did.
* **A character with no stand-in** —a katakana, a symbol outside
  Windows-1252— still leaves its page out, as until 2026-10-16 any letter
  outside ASCII did.

## Why

Until 2026-10-16 a box with a letter the font did not have left its whole
page out. It was chosen on 2026-10-05, when the only such letter known was
Sunset over Imdahl's `ô`: one page out was better than stopping the
conversion. Measured on the nine games, it was most of the talk of the ones
in Spanish and German —950 map pages of Zounds and 166 of Vampires Dawn—,
and it was where Zounds' intro stopped, on a talk with `é`, `¿`, `¡` and
its hero's `ë`. taro's rule for the ports is to unblock the whole game
first, so the stand-ins came first, and the glyphs after, the same day.

Each letter carried takes one value of the pair table, so a game with
letters writes its text with a few pairs fewer, and its window's tables
grow by the letters' glyphs; the cap of 64 leaves the pairs most of their
values.

The text screens waited, at first, because each of their letters was a
tile; since 2026-10-03 they compose their lines into pairs of tiles from a
pool, as the message window does, and since 2026-11-20 every writer draws
from one font, so a letter costs them only its glyph, 832 bytes, when the
boxes do not carry it already: Zounds' `Á` and Vampires Dawn's `Ä`, the
only two, measured on the nine games —see the task below—. A cartridge
whose screens write no letter carries none for them, and is byte for byte
what it was before. Until 2026-11-02 it was 32, the composer's: a letter's index had
to stay below 128, because the second letter of a pair waited in
`FAR_MSG_SECOND` with bit 7 set, and Zounds' 18 letters and the halves of
its icons did not fit. It waits plus one now.

## How it applies

The stand-ins are `plain` in `rm2k-snes-text`, and the glyphs the last six
rows of `low-variable.png`, derived from EasyRPG's `rmg2000` by the rules of
the rest of the sheet —see the fonts' README in `rm2k-snes-text`—. A font
says which it draws with `Font::latin_letters`, and the converter hands them
to the event compiler with the message window. The compiler's first pass
draws all of them and counts which the boxes use
(`Messages::letters_used`); the second draws those —`Window::carrying`—, and
`Messages::store` stands the plain ones in for the rest before trimming the
line. The letters go to the screens stage in `messages.letters`, whose
glyphs `font_bytes_with` puts after the console's 96 in the message
window's tables, and the pairs start after them, `MESSAGE_PAIR_FIRST`.
The text screens' letters are counted by the converter's `letters.rs`
over the texts the cartridge composes at run time —the actors' names,
the items, the skills with their messages, the enemies, the conditions
with their messages, the terms, a 2003 game's battle commands, the
project's `[battle.terms]` and the `ChangeHeroName` names—, leaving out a
text with a cp1252 hole, which is a Shift-JIS leftover. The screens stage
puts them after the boxes' —`texts::ScreenLetters`—, with the blank at
glyph 96 even without icons, since the text layer takes that glyph for a
digit's blank; and one rule, `rm2k_snes_text::plain::screen_text`, decides
what a screen writes as it is and what as its stand-in, in the screens
stage's `item_text`, the battle's and the mapdata stage's `Texts`. A name
a box writes is the screens' text, so `msg_name_letter`, in the runtime's
`actor_names.s`, takes its letters back to the window's numbering, and
the window's tables carry the letters only the screens write too. Until
2026-11-22 the screens wrote every letter as its stand-in; and until the
same morning the battle's texts went raw, and each accent was a `?`:
Zounds' "Látigo de Seda" read "L?tigo de Seda". Tasks:
[`zounds-skill-names-show-a-question-mark-for-an-accent`](https://github.com/lautarodragan/the-advocate-work/blob/main/tasks/zounds-skill-names-show-a-question-mark-for-an-accent.md)
and
[`the-text-screens-draw-latin-1`](https://github.com/lautarodragan/the-advocate-work/blob/main/tasks/the-text-screens-draw-latin-1.md).
Tested by the test game's `test_plain_letters.py`, a box with letters drawn
and letters plain, and its `test_screen_letters.py`, an actor's name and
an item's with letters on the items screen, a shop and a battle, and the
name in a box; and the ports' `test_zounds.py`, the talk with the guards
on map 8 that goes on to map 9, and `test_zounds_battle_text.py`, a hit's
lines in the game's words. Task:
[`latin-1-letters-stop-a-box`](https://github.com/lautarodragan/the-advocate-work/blob/main/tasks/latin-1-letters-stop-a-box.md).
