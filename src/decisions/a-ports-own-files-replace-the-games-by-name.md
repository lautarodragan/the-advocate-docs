# A port's own files replace the game's, by name

## What

**A port can carry files of its own that take the place of the game's**:
`patches/manual/files/`, in the game's project, is RM2k's own tree
—`Picture/`, `System/`, `CharSet/`…— and a file there replaces the game's
with **the same folder and the same name without extension**, without
regard to case. Since 2026-10-19, taro's: Frozen World's menu, three
pictures that did not read on the console, was the first.

```text
frozen-world/patches/manual/files/Picture/MenuEquipPlates.png
```

replaces the game's `Picture/MenuEquipPlates.png`, and would replace a
`picture/menuequipplates.bmp` just the same: RM2k looks a resource up by its
name, without the extension and without minding case, so leaving both would
make which one is found depend on the order of a directory listing.

**A file of ours has to replace something**: in the game or, if the game
lacks it, in the RTP, which is looked up after the game. A misspelled name,
a folder that is not one of RM2k's, a file at `files/`'s root, or two of
ours with one name stop the build —the same rule as a forked map that does
not exist—. And a ChipSet the project *adds*, whose image is built from a
recipe (A Blurred Line's `chipsets/`), may not take a name the game —with
our files— or the RTP already has.

**The build says what it replaced**: `converter.log`, next to the ROM,
prints "The port's own files, in front of the game's" with one line each.

## Why

**The game is not edited**: it is read from its `.zip`, untouched, and
what the port changes goes through its patches. Until this, the patches
could change the game's data —forked maps, added ChipSets— but not its
files: a picture that had to be redrawn had nowhere to go.

**`files/` and not RM2k's folders straight in `manual/`**: the lowercase
folders of `manual/` are our own formats —`forks/`, `new/`, `chipsets/`—,
and a `chipsets/NNNN.json` is a database record with a recipe, not a
ChipSet image. A `ChipSet/` next to it would read as the same thing.

**It is done in one place**, `Environment::unpack` in `rm2k-project`,
which already turned a `.zip` game into a directory: with `files/`, it
builds the game with ours on top —links to the game's files, ours copied—
in `own-files/` in the project's cache, named by the game and by our files'
names and contents, and from then on `--game-dir` is that directory, for
every tool and for the ones it runs. The lookups are several, in several
processes —see [The pipeline](../internals/pipeline.md)—, and none of them
has to know.

**The added ChipSets stay apart**: their images are built by the converter
from the dump's forks, after the game is merged, and the merged directory
is shared by every process and named by its content, so writing into it
would make it mutable. Since one goes first in the search, the build stops
when both would name the same file.

## Pictures drawn for the console

A picture tied to the screen is scaled from RM2k's 320x240 to the console's
256x224, and **text drawn one pixel wide does not survive four fifths**:
redrawing Frozen World's menu flat at RM2k's size blurred again. So a
port's own picture can be drawn at the console's resolution, and the
project names it:

```toml
[graphics]
console_pictures = ["MenuEquipPlates", "MenuItemManage", "MenuPartyStats"]
```

It is shown unscaled, with its centre where RM2k's lands: the runtime
places a picture by its centre and half its size in RM2k's pixels, so the
converter gives it the size RM2k would scale to it —88x56 is 110x60— and
the corner comes out the same on both sides. No runtime change, in the
sprites and in the big pictures' layer alike. A zoom, or a picture fixed to
the map, would scale it, and is refused. In RPG_RT such a picture would
look small: it is the console's, not the game's.

## How it applies

- `rm2k-snes/project/src/own_files.rs` in the-advocate, and
  `Environment::unpack`.
- The cartridges' fingerprint takes `files/` whole, each file with its
  name: changing or renaming a picture rebuilds.
- `[graphics] console_pictures`: `Cropped` in
  `rm2k-snes/convert/src/pictures.rs`, and `Screens::place_unscaled` in
  `rm2k-snes/images/src/show_picture.rs`.
- The first use, and why: [The Frozen World](../ports/frozen-world/README.md);
  the task,
  [`frozen-world-a-legible-menu-from-our-own-pictures`](https://github.com/lautarodragan/the-advocate-work/blob/main/tasks/frozen-world-a-legible-menu-from-our-own-pictures.md).
