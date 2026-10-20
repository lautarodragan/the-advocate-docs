# Zounds

taro's own game, unfinished, made in **RPG Maker 2003**: the first 2003
game here, and the one the tools learned 2003 from. The study that
preceded the port is
[Zounds, an RPG Maker 2003 game](../../research/2026-10-15-zounds-an-rpg-maker-2003-game.md);
the port's project is `zounds/` in the-advocate-ports, built on 2026-10-15,
**without MSU-1**.

## What is 2003 about it, and what the port does with it

* **Ditto reads it whole.** The 2003 fields —classes, battle commands,
  battler animations, the second timer, the turn and command conditions of
  a troop page— and the 2003 bits of the flag fields, which until that day
  came back short: see point 5 of Ditto's README in the-advocate, and the
  tasks
  [`ditto-does-not-read-the-2003-fields`](https://github.com/lautarodragan/the-advocate-work/blob/main/tasks/ditto-does-not-read-the-2003-fields.md)
  and
  [`ditto-drops-the-2003-flag-bits`](https://github.com/lautarodragan/the-advocate-work/blob/main/tasks/ditto-drops-the-2003-flag-bits.md).
* **Its battles are fought with the 2000 battle**: turns ordered by
  agility instead of the ATB, the front view instead of its RTP side-view
  battlers, Attack, Skill, Defend, Item and Escape for everyone. The two
  2003 commands its troop pages carry are compiled for it:
  `CallCommonEvent`, the steal mechanic, as the called body pasted into the
  page; and `ForceFlee`, the game's own Escape command, as an escape that
  cannot fail. The 36 pages that carry the second are conditioned on that
  command being chosen, which the 2000 battle has no event for, so they
  never run here and the built-in Escape stands in —with RPG Maker 2000's
  chance of failing, which the game's own never had—. Task:
  [`the-2003-battle-commands-callcommonevent-and-forceflee`](https://github.com/lautarodragan/the-advocate-work/blob/main/tasks/the-2003-battle-commands-callcommonevent-and-forceflee.md).
* **Its images were XYZ**, RPG Maker 2003's own format, which the tools do
  not read on purpose: the port builds from a branch of the game's
  repository where they are PNG, converted once with EasyRPG's `xyz2png`.
  Task:
  [`the-tools-do-not-read-xyz-images`](https://github.com/lautarodragan/the-advocate-work/blob/main/tasks/the-tools-do-not-read-xyz-images.md).

## What the game lacks, and the port fakes

The repository references 197 assets it never had —the RPG Maker 2003
RTP, the 2000 RTP under other RTPs' names, and 55 that are nowhere—. On
the port's branch each name has a stand-in: six songs that are A Blurred
Line's, with the same name, are the real ones; seven more were found again
by name in a public MIDI archive, for a private build only; twenty are the 2000 RTP's
own files, found through EasyRPG's RTP table; 35 of the RPG Maker 2003
RTP's are EasyRPG's free RTP (CC BY 4.0) under the game's names, since
2026-10-16 —EasyRPG has 35 of the 95 the game uses: six charsets, two
chipsets, two facesets, a panorama, the System2 and 23 sounds—; the other
136 are a copy of some file of the game from the same folder, so a missing
chipset is some chipset and a missing sound some sound. The debug
enemy's numbers were
brought within sixteen bits. Task:
[`zounds-the-repository-lacks-197-assets`](https://github.com/lautarodragan/the-advocate-work/blob/main/tasks/zounds-the-repository-lacks-197-assets.md).

## The debug map

**"New Game" starts in a room of the port's own**, map 1000 of
`zounds/patches/manual/maps/`, so the parts being tested are reached
without replaying the way there: it costs one menu option and nothing
else. "Continue" is untouched. Its four doors are the gaps in the bottom
wall, and each asks before jumping:

* **The game**: map 1, from its start, as the game without the room.
* **The ship, right after the intro**: Uname's room, map 17, where the
  first dream leaves her, in the guard's uniform and with the intro's gold
  and permit.
* **The desert, the first time**: map 34, where Kazar's talk in the
  download room leaves her, with everything the ship sets on the way —the
  boss taking the gold, the talk at the cafeteria, Yôzar's amulet, the
  desert's explanation, the second dream—. Kazar joins there, through the
  game's own scene.
* **The desert with the pickaxe**: the same place, with Kazar, after
  Bandurion's explanation and his pickaxe.

Each door sets what the game's pages set on the way to that point, in
their order, with the constants of map 1's settings page; the comments of
the door's page say which page each thing comes from. What the player may
or may not do on the way —chests, shops, Iman's flashback— is not done,
except Yôzar's amulet, which the game stops offering once Kazar joins.
The room puts Kazar in the party; the doors to a place where the game
does not have him yet take him out.

**The man in the room levels Uname and Kazar up**, one level or five at
a time, as many times as wanted.

`[debug] debug_map = false` in `rm2k-snes.toml` takes the room out for a
clean delivery: "New Game" starts on map 1, and the ROM comes out byte for
byte as without the room. Task:
[`zounds-a-debug-map`](https://github.com/lautarodragan/the-advocate-work/blob/main/tasks/zounds-a-debug-map.md).

## Known issues

What playing the port shows today that the original does not. It is kept
current: whoever fixes one removes its line in the same change.

* **Most picture-made screens do not show**: the notes and the pick-axe
  screen (common events 21, 27, 29, 40 and 41), which are common events
  compiled apart from the maps and so without their small pictures —the
  notebook (29) shows its background alone, over a hidden map—; the menu on map
  136, whose two faces and bar do not fit the sprite tiles; and the skill
  screen on map 190, whose box and list are big pictures that hide the
  small ones over them. The race's score board on map 75 shows all its
  numbers since 2026-10-16. Drafts:
  [`pictures-in-common-events-compiled-apart-are-not-shown`](https://github.com/lautarodragan/the-advocate-work/blob/main/tasks/pictures-in-common-events-compiled-apart-are-not-shown.md),
  [`zounds-menu-pictures-on-map-136-need-512-tiles`](https://github.com/lautarodragan/the-advocate-work/blob/main/tasks/zounds-menu-pictures-on-map-136-need-512-tiles.md),
  [`small-pictures-under-a-big-one-are-hidden`](https://github.com/lautarodragan/the-advocate-work/blob/main/tasks/small-pictures-under-a-big-one-are-hidden.md).
  The intro itself goes by: the credits on map 106 for five seconds,
  then the action key takes it to the continent (107) and the cutscenes on
  maps 1, 4, 5 and 6, until the player walks on map 7, since 2026-10-16;
  and the guard's scene there goes up the stairs to map 8 with the screen
  back, and the talk with the two guards there on to map 9, since the same
  day. Task:
  [`zounds-pictures-and-the-picture-made-screens`](https://github.com/lautarodragan/the-advocate-work/blob/main/tasks/zounds-pictures-and-the-picture-made-screens.md).
* **In the menu, the shops and the battle the accents are lost**: the text
  screens draw ASCII, so the hero is "Uname" there and an item's
  "Líquido" is "Liquido". The message boxes draw them. See
  [Letters the font does not have](../../divergences/letters-the-font-does-not-have.md).
* **The stand-in assets**: wrong chipsets on the maps that asked for the
  RTP's or the lost ones, one sound for 90 missing sounds, one song for
  the three songs still missing (`waltz`, `boat1`, `boat2`); and the
  EasyRPG RTP's charsets, faces and sounds look and sound unlike RPG Maker
  2003's. See above.
* **The dream has no fog**: on map 30, after sleeping in Uname's bed, the
  reddish fog the dream shows over the screen is missing, because it is
  shown at 110% and the console only shrinks pictures. The dream itself
  plays —the boss runs off, and two minutes later the story goes back to
  the room— since 2026-10-17; until then the map was left out. Draft:
  [`zounds-the-dreams-fog-is-not-shown`](https://github.com/lautarodragan/the-advocate-work/blob/main/tasks/zounds-the-dreams-fog-is-not-shown.md).
* **The Kuak-Man mini-game has 108 of its 314 coins**: map 178 has more
  events than the console's 120, and the coins with the highest ids are
  left out. Draft:
  [`zounds-kuak-man-loses-its-coins`](https://github.com/lautarodragan/the-advocate-work/blob/main/tasks/zounds-kuak-man-loses-its-coins.md).
* **The throwing items that poison, paralyze or burn only hurt**: since
  2026-10-19 the battle offers the special items —Kazar throws the
  Piedra—, and Veneno, Paralizador, Pétalo Violeta, Ácido, Ala Eléctrica
  and Bola de Fuego do their damage and give no condition, which no skill
  does in battle yet. The battle writes only the item's line, without the
  damage. See
  [What items in battle do not model](../../divergences/what-items-in-battle-do-not-model.md).
  Drafts:
  [`items-that-give-a-condition-in-battle-only-hurt`](https://github.com/lautarodragan/the-advocate-work/blob/main/tasks/items-that-give-a-condition-in-battle-only-hurt.md),
  [`a-skill-in-battle-writes-no-result-line`](https://github.com/lautarodragan/the-advocate-work/blob/main/tasks/a-skill-in-battle-writes-no-result-line.md).
* **Without MSU-1 four `.wav` songs are silent**: the battle and victory
  themes, the mountains' (map 33) and the small caves' (maps 89, 91, 92
  and 177), which the map tree sets; no MIDI of them was found. The title
  theme plays a fan MIDI of it. With MSU-1 all five play from their
  tracks. Draft:
  [`zounds-four-wav-songs-without-a-midi`](https://github.com/lautarodragan/the-advocate-work/blob/main/tasks/zounds-four-wav-songs-without-a-midi.md).
* **A question waits for A before its options show**: when a message is
  followed by a choice that fits under it —Kazar's "¿Has entendido?" the
  first time in the desert—, RPG Maker draws both in one box; here the
  message comes first, alone, and the options open in a box of their own.
  Draft:
  [`a-message-and-the-choice-after-it-share-a-box-in-rm2k`](https://github.com/lautarodragan/the-advocate-work/blob/main/tasks/a-message-and-the-choice-after-it-share-a-box-in-rm2k.md).
* **No boat**: the fishing game's vehicle. Draft:
  [`vehicles-enterexitvehicle-and-the-boat`](https://github.com/lautarodragan/the-advocate-work/blob/main/tasks/vehicles-enterexitvehicle-and-the-boat.md).
* **The maps that wrap around stop at their edges**: on Passage 1, map
  70, the corridors that go on across the map's edges end there, and
  the door to Cuarto 2 (103), reached only that way, cannot be reached
  walking; on Scape Way (150) the hero takes one step instead of
  walking up forever while the party talks —that one also needs his
  route to repeat—. The dream
  (30) and the Secret Entrance (137) also wrap. See [Maps that wrap
  around](../../research/2026-10-19-maps-that-wrap.md). Task:
  [`maps-that-wrap-around`](https://github.com/lautarodragan/the-advocate-work/blob/main/tasks/maps-that-wrap-around.md).
