# The manual patches are one folder per kind of asset

## What

**`patches/manual/`, in a game's project, is laid out by kind of asset,
flat**, so a person browsing it finds everything about a ChipSet, a
picture or a map in one place. Since 2026-10-19, taro's:

```text
patches/manual/
  ChipSet/        an image replaces the game's of the same name; a
                  NNNN.json adds the database record of that id, with the
                  recipe of its image
  Picture/, CharSet/, System/…
                  RM2k's own folders: each file replaces the game's of the
                  same folder and name
  maps/           MapNNNN.json and MapNNNN.tiles, as in Ditto's data/maps/
  commonevents/   NNNN.json: a common event of the game, redone
  README.md       what is there, and why
```

**RM2k's folders keep RM2k's names, and what RM2k keeps inside its map and
database files takes the name Ditto's dump gives it**: `maps/` is
`data/maps/`, and `commonevents` is the section `data/db/commonevents.json`
holds. The capitalised folders are the game's; the lowercase ones, ours.

**A map's id says what it is.** An id of **1000 or more is a new map**,
which brings its node of the tree in its `tree` field, since the game has
none for it, and may not take an id the game uses. **Below 1000 it replaces
the game's map**, keeps the game's node, and replacing a map the game does
not have is an error. A Blurred Line's debug map, the only new map, is
1000.

**A recipe and a hand-made image never share a name**: a
`ChipSet/NNNN.json` whose image is called like an image lying in
`ChipSet/` stops the build. Only `ChipSet/` takes records today; a
`NNNN.json` in another of RM2k's folders stops it too.

`patches/auto/` stays as it is: it is the converter's decisions, not the
game's files.

## Why

**Until then the folders were one per kind of patch**: `forks/` for the
game's maps redone, `new/` for the maps the game lacks, `chipsets/` for
the added ChipSet records, `files/<Folder>/` for the files that replace
the game's, and `commonevents/`. A ChipSet image of ours and a ChipSet
record of ours lived in two folders, and a map in one of two depending on
whether the game had it.

**What the two map folders guarded is kept by the id**: a fork with a
misspelled number must not pass for a new map, nor a new map cover one of
the game's without warning. With a range of its own for new maps, a typo
on either side is still an error. A thousand is far above any map a game
we know of numbers —RM2k goes up to 9999—, and reads as "ours" at a
glance.

**A record and an image of one name would cover each other without a
word**: the recipe's image is built after the game is merged with the
port's files, and the built images go first in the search. One of the two
is a mistake, so the build says so where both files are.

## How it applies

- The RM2k folders: `own_files.rs` in `rm2k-project`, in the-advocate,
  with its `OURS` —the folders that are not files of the game— and
  `is_record`; see [A port's own files replace the game's, by
  name](a-ports-own-files-replace-the-games-by-name.md).
- The maps, the records and the common events: `fork.rs` in
  `rm2k-snes-convert`, with `FIRST_NEW_MAP` and its tests.
- How each stage gets them: [The pipeline](../internals/pipeline.md).
- The task,
  [`manual-patches-one-folder-per-kind-of-asset`](https://github.com/lautarodragan/the-advocate-work/blob/main/tasks/manual-patches-one-folder-per-kind-of-asset.md);
  each old path is in `RENAMES.md` at the root of the-advocate-ports.
