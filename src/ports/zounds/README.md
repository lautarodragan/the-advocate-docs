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
  battlers, and the same five commands in the same order for everyone,
  named since 2026-10-20 by each actor's own battle commands —Atacar,
  Técnica, Defender, Objeto or Kazar's Tirar-Usar, Escapar—; until then
  they were the 2000 terms, two of them the editor's Japanese. See
  [A 2003 game's battle commands](../../divergences/a-2003-game-s-battle-commands.md).
  Task:
  [`a-2003-game-s-battle-commands-are-its-own`](https://github.com/lautarodragan/the-advocate-work/blob/main/tasks/a-2003-game-s-battle-commands-are-its-own.md).
  The two
  2003 commands its troop pages carry are compiled for it:
  `CallCommonEvent`, the steal mechanic, as the called body pasted into the
  page; and `ForceFlee`, the game's own Escape command, as an escape that
  cannot fail. The 36 pages that carry the second are conditioned on that
  command being chosen, which the 2000 battle has no event for, so they
  never run here and the built-in Escape stands in —with RPG Maker 2000's
  chance of failing, which the game's own never had—. Task:
  [`the-2003-battle-commands-callcommonevent-and-forceflee`](https://github.com/lautarodragan/the-advocate-work/blob/main/tasks/the-2003-battle-commands-callcommonevent-and-forceflee.md).
  And the 2000 battle writes a line for each hit —`Kazar ataca.` over
  `Pez Shark 7 HP perdidos.`—, which the 2003 one never did, so the game's
  database holds the editor's Japanese in those terms: the port's own
  Spanish ones are in `[battle.terms]` of its `advocate.toml`, since
  2026-10-19. See
  [A 2003 game's battle writes the 2000 lines](../../divergences/a-2003-game-s-battle-writes-the-2000-lines.md).
  Task:
  [`zounds-battle-messages-cannot-be-read`](https://github.com/lautarodragan/the-advocate-work/blob/main/tasks/zounds-battle-messages-cannot-be-read.md).
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
chipsets, two facesets, a panorama, the System2 and 23 sounds—; one,
`NAU_Splash`, is a CC0 splash from OpenGameArt, since 2026-10-20 (see
below); the other 135 are a copy of some file of the game from the same
folder, so a missing chipset is some chipset and a missing sound some
sound. The debug
enemy's numbers were
brought within sixteen bits. Task:
[`zounds-the-repository-lacks-197-assets`](https://github.com/lautarodragan/the-advocate-work/blob/main/tasks/zounds-the-repository-lacks-197-assets.md).

## Its icons come from `RPG_RT_a.exe`

The items' and skills' names start with an ExFont icon —`$NPoción Roja`—
and their descriptions count on them: "+35$N" is "+35 HP". Those icons
—HP, MP, attack, defense and the items' own pictures— are the ExFont of the
`RPG_RT_a.exe` that came with the repository, a Spanish-translated RPG
Maker 2003 runtime that does not load the game: it looks for other
signatures in the data files. Its `RPG_RT.exe`, the one that runs the
game, carries RPG Maker's stock one, the zodiac. The port keeps the first
as a sheet, `exfont.png`, extracted once on 2026-11-02, and the project
file names it, `[text] exfont = "exfont.png"`: the tools do not read a
game's executable. See [What ExFont icons do not
model](../../divergences/what-exfont-icons-do-not-model.md).

## What is fixed in the game itself

The game is taro's and its repository can be retouched, so a bug that is
the game's and not the port's is fixed there, and RPG Maker 2003 plays the
fix too:

* **The doorways of the cave's Arriba, 2 and 3** (maps 79 and 80), into
  their little rooms, were touch events on the layer below the hero,
  standing on the doorway's tile, which closes its cell: walking into them
  did nothing, in RPG Maker 2003 and EasyRPG as on the console. They are on
  the hero's layer since 2026-10-19, as the same doorway on map 63 always
  was. Task:
  [`zounds-two-events-in-the-cave-do-nothing`](https://github.com/lautarodragan/the-advocate-work/blob/main/tasks/zounds-two-events-in-the-cave-do-nothing.md).
* **The dungeon under the crack plays its own song.** None of its maps
  named one: they all said "the parent's" up to map 33, Mountains, and
  RPG Maker climbs the tree on every map change, also over a song an event
  started, so past map 59, whose autostart page plays `Zounds_Dungeon_01`,
  the cave played `18 - Desert flower` —in RPG Maker 2003's own runtime as
  on the console—. Since 2026-10-20 the four folders that hold the dungeon
  under map 58 (Caída) —68, 69, 98 and 100— name `Zounds_Dungeon_01`, and
  every map under them inherits it; 59, where the fall lands, says "the
  event's", so the fall's `ff3boss` goes on until 59's page fades it and
  plays the dungeon's song. 58 itself, the way out (105) and the outdoor
  map and passage under the dungeon (81 and 189) keep Mountains' song, and
  so do 51 to 53, outside the crack. Task:
  [`zounds-the-cave-plays-the-mountains-song`](https://github.com/lautarodragan/the-advocate-work/blob/main/tasks/zounds-the-cave-plays-the-mountains-song.md).
* **The spelling of its Spanish text.** What the player reads —messages,
  choices, the database's names, descriptions and terms— carried hundreds
  of spelling mistakes: *save* for *sabe*, *deve* for *debe*, *vever* for
  *beber*, missing accents and opening `¿` and `¡`, *Hay* for *Ay*. Since
  2026-10-20 they are fixed in the game's data, and the author's voice
  stays: voseo, slang, the guards' parody of Spanish from Spain, the
  invented languages, the names. What looked like it could be on purpose
  was left as it was, and listed for taro. Task:
  [`zounds-spelling`](https://github.com/lautarodragan/the-advocate-work/blob/main/tasks/zounds-spelling.md).
* **The water Kazar hears in the cave is a splash.** On map 60, under the
  crack, a parallel event plays `NAU_Splash` in pairs and Kazar says he
  heard water; map 71 plays it too. The repository never had the file,
  and its stand-in was `Close1`, a door. Nothing was found by that name,
  and Don Miguel's RTP `Water` sounds carry no free license, so since
  2026-10-20 it is a CC0 splash, rubberduck's `splash_02` from
  OpenGameArt's "40 CC0 water / splash / slime SFX", credited in the
  game's `CC0-sounds/`. It is not the sound the game had. Task:
  [`zounds-map-60s-water-sounds-like-a-door`](https://github.com/lautarodragan/the-advocate-work/blob/main/tasks/zounds-map-60s-water-sounds-like-a-door.md).
* **Three messages whose control codes were broken.** The farmer's job
  offer on map 139 opened with `\c[6]Trabajo\[4]`, a colour code without
  its letter, where the same question on his other page says
  `\c[6]Trabajo\c[4]`; Madamme Lessier's laugh on map 161 began with
  `\v[20]`, which writes a variable's value in front of it, where every
  other slowed laugh or silence in the game is `\s[20]`; and Unamë's
  `¿No nos vas a pagar?\` on map 24 ended in a lone backslash, where the
  game's pause, `\|`, closes hundreds of lines. Since 2026-10-20 they say
  `\c[4]`, `\s[20]` and `\|`. Task:
  [`zounds-three-messages-with-broken-control-codes`](https://github.com/lautarodragan/the-advocate-work/blob/main/tasks/zounds-three-messages-with-broken-control-codes.md).
* **Three messages name their item as the database does.** Unamë stole
  *Grasa de Camello*, the chef's menu offered *Puré con Papas* and the
  fishing score counted *Kinshi*, for the item Grasa de Camaro, the item
  Puré de Papas and the fish Kinsh. taro decided the item's name wins;
  since 2026-10-20 the messages say it. Task:
  [`zounds-three-messages-name-an-item-otherwise`](https://github.com/lautarodragan/the-advocate-work/blob/main/tasks/zounds-three-messages-name-an-item-otherwise.md).
* **Two secret chests shared a switch.** The secret chest on MiniCave 177
  is a copy of MiniCave 89's and kept its switch, Secret- Cofre 1, so
  opening either showed the other open, without its items. Since
  2026-10-20 177's chest has a switch of its own, Secret- Cofre 4, one the
  game never used. Task:
  [`zounds-two-chests-share-a-switch`](https://github.com/lautarodragan/the-advocate-work/blob/main/tasks/zounds-two-chests-share-a-switch.md).
* **Map 81 leads back to the crack.** The crack at the top of map 79
  lands the party on map 81 above a flight of stairs, and the map's only
  exit was on the left, to 189: nothing went back to 79, so the shortcut
  led nowhere from the cave's side. Since 2026-10-20 two touch events
  halfway down the stairs, one per cell of their width, take the party
  back to 79 just below the crack, facing away from it, with the cave's
  tint and song, as 79's own exit does the other way. Task:
  [`zounds-no-way-back-from-map-81`](https://github.com/lautarodragan/the-advocate-work/blob/main/tasks/zounds-no-way-back-from-map-81.md).

## The debug map

**"New Game" starts in a room of the port's own**, map 1000 of
`zounds/patches/manual/maps/`, so the parts being tested are reached
without replaying the way there: it costs one menu option and nothing
else. "Continue" is untouched. Its eight doors are the gaps in the bottom
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
* **The first dungeon**: the cave under the crack, at the top of the
  stairs down from its entrance (map 58), with everything on the way from
  the pickaxe —the rock into the crack broken with it, Kazar's notebook
  and its eight notes, Bandurion's "be careful"—. A few steps down, the
  game's own fall drops the party into the cave.
* **The end of the Death Desert's maze**: map 113, where the maze's last
  jump leaves the party, by the mouth of the desert's cave, with everything
  from the first dungeon's door on —the crack's girl and her fossil, the
  ship gone, the fossil traded to Bandurion for the Talisman the desert
  asks for—.
* **Delmar** and **the town of Itân**: maps 183 and 157, which the story
  does not reach —only the developer's test map leads there—, where that
  map's jumps land. Nothing there reads the story, so these two doors set
  only the game's settings and keep the room's party.

Each door sets what the game's pages set on the way to that point, in
their order, with the constants of map 1's settings page; the comments of
the door's page say which page each thing comes from. What the player may
or may not do on the way —chests, shops, Iman's flashback— is not done,
except Yôzar's amulet, which the game stops offering once Kazar joins.
The room puts Kazar in the party; the doors to a place where the game
does not have him yet take him out.

**The man in the room levels Uname and Kazar up**, one level or five at
a time, as many times as wanted.

`[debug] debug_map = false` in `advocate.toml` takes the room out for a
clean delivery: "New Game" starts on map 1, and the ROM comes out byte for
byte as without the room. Tasks:
[`zounds-a-debug-map`](https://github.com/lautarodragan/the-advocate-work/blob/main/tasks/zounds-a-debug-map.md),
[`zounds-a-debug-door-to-the-first-dungeon`](https://github.com/lautarodragan/the-advocate-work/blob/main/tasks/zounds-a-debug-door-to-the-first-dungeon.md)
and
[`zounds-a-debug-door-to-the-desert-jail`](https://github.com/lautarodragan/the-advocate-work/blob/main/tasks/zounds-a-debug-door-to-the-desert-jail.md).

## Walking through everything

**Select walks the hero through walls and events**, and again stops it:
RPG_RT's test play does it while Ctrl is held. While it is on the hero
blinks, and there are no random encounters. It is a debug knob,
`[debug] phasing` in `advocate.toml`, and off the ROM comes out byte for
byte as without it. How it differs from RPG_RT: [Phasing is a toggle on
Select](../../divergences/phasing-is-a-toggle-on-select.md). Task:
[`a-debug-toggle-to-walk-through-everything`](https://github.com/lautarodragan/the-advocate-work/blob/main/tasks/a-debug-toggle-to-walk-through-everything.md).

## Known issues

What playing the port shows today that the original does not. It is kept
current: whoever fixes one removes its line in the same change.

* **Most picture-made screens do not show**: the pick-axe screen and the
  notebook (common events 21, 27, 29, 30 and 41), common events compiled
  apart from the maps whose small pictures no map leaves room for beside
  its characters —the notebook (29) shows its background alone, over a
  hidden map—; the notes (40) and the permit (6) show only on the maps that
  leave room for theirs; and the menu on map 136, whose two faces and bar
  do not fit the sprite tiles. The race's score board on map 75 shows all
  its numbers since 2026-10-16, and the skill screen on map 190 its panels,
  list, selector and face since 2026-10-20 —without the selector's
  half-transparent fill, which the port leaves out: see its
  `patches/manual/README.md`—. Drafts:
  [`zounds-pick-axe-and-notebook-fit-beside-no-map`](https://github.com/lautarodragan/the-advocate-work/blob/main/tasks/zounds-pick-axe-and-notebook-fit-beside-no-map.md),
  [`zounds-menu-pictures-on-map-136-need-512-tiles`](https://github.com/lautarodragan/the-advocate-work/blob/main/tasks/zounds-menu-pictures-on-map-136-need-512-tiles.md).
  The intro itself goes by: the credits on map 106 for five seconds,
  then the action key takes it to the continent (107) and the cutscenes on
  maps 1, 4, 5 and 6, until the player walks on map 7, since 2026-10-16;
  and the guard's scene there goes up the stairs to map 8 with the screen
  back, and the talk with the two guards there on to map 9, since the same
  day. Task:
  [`zounds-pictures-and-the-picture-made-screens`](https://github.com/lautarodragan/the-advocate-work/blob/main/tasks/zounds-pictures-and-the-picture-made-screens.md).
* **Unamë's face on the skill screen has orange hair**, not green: a face
  shown as a picture keeps its fifteen most used colours, and they are all
  her skin's. Draft:
  [`a-small-picture-s-fifteen-colours-lose-a-whole-hue`](https://github.com/lautarodragan/the-advocate-work/blob/main/tasks/a-small-picture-s-fifteen-colours-lose-a-whole-hue.md).
* **In the menu, the shops and the battle the accents are lost**: the text
  screens draw ASCII, so the hero is "Uname" there and an item's
  "Líquido" is "Liquido". The message boxes draw them. See
  [Letters the font does not have](../../divergences/letters-the-font-does-not-have.md).
* **A skill's name with an accent carries a `?` in the battle's text**:
  "L?tigo de Seda", "P?talo Violeta", where the items read "Pocion" —found
  in the cartridge's text, not looked at on the screen yet—. Draft:
  [`zounds-skill-names-show-a-question-mark-for-an-accent`](https://github.com/lautarodragan/the-advocate-work/blob/main/tasks/zounds-skill-names-show-a-question-mark-for-an-accent.md).
* **The boxes shown in battle have no icons**: an item's icon in a box a
  troop page shows —the prize's "Objeto Especial Veneno"— is left out, as
  its accents are. Everywhere else they show. See [What ExFont icons do not
  model](../../divergences/what-exfont-icons-do-not-model.md). Draft:
  [`a-troop-page-s-box-drops-its-icons`](https://github.com/lautarodragan/the-advocate-work/blob/main/tasks/a-troop-page-s-box-drops-its-icons.md).
* **The speakers' names and the other coloured words have fewer shades
  than RM2k's gradient, and a second colour in the same box comes out in
  the window's own blues** —"¡Energía y Magia totalmente recuperados!",
  with Magia not in its green—; and the message window has one colour
  less: its last entry is the box's ink. See [What message escapes do not
  model](../../divergences/what-message-escapes-do-not-model.md#the-colour).
* **The stand-in assets**: wrong chipsets on the maps that asked for the
  RTP's or the lost ones, one sound for 89 missing sounds, one song for
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
* **Teleport is not in the battle's skill list**: once the party learns
  it, RPG Maker lists it in battle, grey, and the port leaves it out.
  Draft:
  [`the-battle-leaves-teleport-and-escape-skills-out-of-the-list`](https://github.com/lautarodragan/the-advocate-work/blob/main/tasks/the-battle-leaves-teleport-and-escape-skills-out-of-the-list.md).
* **The battle's commands go in another order**: Unamë's and Kazar's
  window puts Defender before their Objeto or Tirar-Usar, Bandurion has an
  Escapar he does not have in RPG Maker, and Escapar is RPG Maker 2000's,
  which can work at once or fail, where the game's counts its tries
  —"Escapando... 1/3"— and never fails. Draft:
  [`a-2003-actor-s-command-window-is-the-2000-one`](https://github.com/lautarodragan/the-advocate-work/blob/main/tasks/a-2003-actor-s-command-window-is-the-2000-one.md).
* **Without MSU-1 four `.wav` songs are silent**: the battle and victory
  themes, the mountains' (map 33) and the small caves' (maps 89, 91, 92
  and 177), which the map tree sets; no MIDI of them was found. The title
  theme plays a fan MIDI of it. With MSU-1 all five play from their
  tracks. Wishlist:
  [`zounds-four-wav-songs-without-a-midi`](https://github.com/lautarodragan/the-advocate-work/blob/main/tasks/zounds-four-wav-songs-without-a-midi.md).
* **A question waits for A before its options show**: when a message is
  followed by a choice that fits under it —Kazar's "¿Has entendido?" the
  first time in the desert—, RPG Maker draws both in one box; here the
  message comes first, alone, and the options open in a box of their own.
  Task:
  [`a-message-and-the-choice-after-it-share-a-box-in-rm2k`](https://github.com/lautarodragan/the-advocate-work/blob/main/tasks/a-message-and-the-choice-after-it-share-a-box-in-rm2k.md).
* **The desert's hot stones hurt in silence**: since 2026-11-01 each step
  onto one takes its HP and flashes red, as in RPG Maker, but without the
  fire sound it plays; and the rocky ground of the mountains (maps 33, 47,
  55 to 57, 81, 87 and 94) has no footsteps. Task:
  [`terrains-do-what-rm2k-does`](https://github.com/lautarodragan/the-advocate-work/blob/main/tasks/terrains-do-what-rm2k-does.md).
* **The dice man on map 156 does not answer**: his game asks for the
  die's number in a box that comes after a label, which the port does not
  draw yet. Draft:
  [`an-input-number-after-a-label-has-no-message`](https://github.com/lautarodragan/the-advocate-work/blob/main/tasks/an-input-number-after-a-label-has-no-message.md).
* **No boat**: the fishing game's vehicle. Draft:
  [`vehicles-enterexitvehicle-and-the-boat`](https://github.com/lautarodragan/the-advocate-work/blob/main/tasks/vehicles-enterexitvehicle-and-the-boat.md).
* **A sign's text blacks out the cave's edges beside it**: on map 96,
  while the sign's message is up, the dark tiles on the map's upper layer
  —the edges and walls past the floor— go black on the scanlines of its
  lines of text, across the whole screen, and come back when it closes;
  on map 86 only the column of the cave's edge next to the floor does.
  And the wooden board RM2k draws under the text is not there. A message
  without a background shares its layer with the map's layer above: see
  the divergence [The window without a background does not cover the
  characters](../../divergences/the-window-without-a-background-does-not-cover-the-characters.md).
  Map 96 keeps its ceiling there because its statue can be pushed through
  walls. Tasks:
  [`a-transparent-message-blacks-out-tiles-on-its-lines`](https://github.com/lautarodragan/the-advocate-work/blob/main/tasks/a-transparent-message-blacks-out-tiles-on-its-lines.md),
  [`above-tiles-no-character-can-reach-go-below`](https://github.com/lautarodragan/the-advocate-work/blob/main/tasks/above-tiles-no-character-can-reach-go-below.md);
  draft:
  [`zounds-map-96-keeps-its-ceiling-above-for-its-statue`](https://github.com/lautarodragan/the-advocate-work/blob/main/tasks/zounds-map-96-keeps-its-ceiling-above-for-its-statue.md).
* **On Scape Way (150) the hero takes one step** instead of walking up
  forever while the party talks: the map wraps around, which the console
  does since 2026-10-19, but his walk is a route that repeats, run once.
  Task:
  [`a-forced-routes-repeat-flag-is-dropped`](https://github.com/lautarodragan/the-advocate-work/blob/main/tasks/a-forced-routes-repeat-flag-is-dropped.md).
* **After a jump the hero faces the way he walked into it**: RPG Maker
  2003 turns him the way the jump asks —up onto Delmar's path from the
  debug room, right onto map 113 from the maze's door—, and 104 of the
  game's jumps ask for one. Draft:
  [`a-teleport-s-facing-is-not-applied`](https://github.com/lautarodragan/the-advocate-work/blob/main/tasks/a-teleport-s-facing-is-not-applied.md).
