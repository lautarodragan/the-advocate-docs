# One command, and no tool of ours runs another

## What

**Building a game is one command a person can type**, `advocate`, without
an AI or a list of manual steps behind it. And two rules shape what is
under it:

* **No binary of ours runs another binary of ours.** The domains —the
  maps, the event compiler, the screens, the pictures, the battle, the
  emission, the music, the patches— are libraries, and so is the
  converter that joins them; `advocate` links them all. What it still runs
  are tools that are not ours: `ca65` and `ld65`, and what the MSU-1
  tracks are synthesized with.
* **The tests consume no binary.** They test the product of the pipeline
  —the ROM, its `.dbg`, the dump, and a file `advocate build` leaves in
  `build/` with what the tools know about the project—, not its parts.
  What a test expects comes from what RPG Maker 2000's rules say, from a
  golden, or from what the ROM shows on its own; never from one of our
  domains asked for the answer. The emulator core the tests play the ROM
  in is a library, and the natural limit: a test needs a ROM someone
  built.

Only what builds a ROM goes in `advocate`: `build`, `assemble`, `space`.
The domain tools a person runs for something else —Ditto, Goombario,
`fxpak`, `rm2k-sketch`, the event compiler's `culprits`— stay commands of
their own.

**It is a direction, not yet the state.** Decided on 2026-11-24 by taro,
after a review of which of our tools started others as processes; written
here before it is done so that it does not live only in the tasks that
carry it out:

* [`cartridges-runs-itself-to-assemble`](https://github.com/lautarodragan/the-advocate-work/blob/main/tasks/cartridges-runs-itself-to-assemble.md)
* [`the-stages-speak-json-nobody-types`](https://github.com/lautarodragan/the-advocate-work/blob/main/tasks/the-stages-speak-json-nobody-types.md):
  the stages become libraries, and the contracts between them Rust types
* [`one-advocate-command`](https://github.com/lautarodragan/the-advocate-work/blob/main/tasks/one-advocate-command.md)
* [`a-stage-s-cache-key-once-it-is-a-library`](https://github.com/lautarodragan/the-advocate-work/blob/main/tasks/a-stage-s-cache-key-once-it-is-a-library.md)
* [`battle-tests-without-simulate`](https://github.com/lautarodragan/the-advocate-work/blob/main/tasks/battle-tests-without-simulate.md)
  and
  [`tests-without-the-stages-subcommands`](https://github.com/lautarodragan/the-advocate-work/blob/main/tasks/tests-without-the-stages-subcommands.md)

Until they land, [The converter's stages](../internals/the-converters-stages.md)
and [The pipeline](../internals/pipeline.md) describe what there is today.

## Why

* **A pipeline one can reason about.** When this was decided a build was
  `rm2k-snes-cartridges`, which runs `rm2k-snes-convert`, which runs each
  stage —a program found on the `PATH`, handed a JSON request in a
  temporary directory—, and then ran itself again to assemble (it calls
  that code since the same day, the first of the tasks above). Which
  binary answers depends on the `PATH`; what goes between them is JSON
  that neither side's compiler checks. As libraries, the contract between
  two domains is a function signature.
* **The stages were programs by history, not by need.** They were small
  tools that compose in the Unix way since the converter was written in
  Python; when it moved to Rust they stayed so, and the stage cache later
  took advantage of it, keying each answer by its binary's hash. Simplifying
  the architecture comes first; a cache key per domain as trustworthy as
  that hash is its own task, and until then every answer is keyed by the
  one binary.
* **A test that asks our code what to expect checks our code against
  itself.** The battle tests demanded the exact HP our own battle model
  gave, which held only while two implementations of ours agreed in every
  detail —the order of each step, each rounding, the same random
  sequence, which is ours and not RPG Maker's—. Change the generator and
  they go red with nothing wrong. A test states what the original game's
  rules say: exact where they are deterministic, a range or an average
  where they roll.

## How it applies

* A new domain is a library the converter calls, not a program it runs;
  a new need of the tests is read from what the build leaves, not asked of
  a domain.
* **`ca65` and `ld65` stay inside `advocate build`**, though they could
  leave it: the converter's job could end at a source tree any compatible
  assembler builds into the ROM, with the build being `advocate` and then
  the two calls. It was left open on purpose —it costs a little of the
  one-command experience, and gains little while there is one assembler—;
  what holds either way is that they are not ours.
