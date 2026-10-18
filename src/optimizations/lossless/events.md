# Lossless: events

## A common event's shared body, called with `OP_CALL`

A common event called from the foreground has one body in the cartridge,
called with `OP_CALL` instead of pasted in place, when the body compiled
with the map is byte for byte the one without it.

- **Saves**: ROM and WRAM —a deep chain of calls pasted in place does not
  fit at all—.
- **Where**: `OP_CALL` in `rm2k-snes/bytecode/src/events.rs` and its
  `op_call` in the interpreter; `refresh` in `compile.rs` of
  `rm2k-snes/events/`.
- **More**: [CallEvent: a shared body, or pasted in place](events/common-event-calls.md).

## `CallEvent` pasted in place, with no interpreter stack

What does not go to a shared body —parallel pages, bodies that depend on
the map, calls to map events— is pasted in place: the callee runs whole
before the caller continues, so no stack of interpreters is needed.

- **Saves**: RAM —no interpreter per call— and the cost of a call in
  parallel pages.
- **Where**: the event compiler, `rm2k-snes/events/`.
- **More**: [CallEvent: a shared body, or pasted in place](events/common-event-calls.md#pasted-in-place).

## Identical pages are stored once

Two pages of a map whose compiled bytecode is the same point at the same
code in the map's block, and so do two identical own routes: the page
table already carries a 24-bit pointer per page. A battle screen made of
one page per monster, all with the same commands, is the case it was made
for.

- **Saves**: WRAM, where the map's bytecode is decompressed; ROM only a
  little, because LZ2 already finds most of the repetition. How much, per
  map: the `map_code` table of the cartridge's `mapdata.inc`.
- **Only within a map**: each map's block is its own stream, decompressed
  alone.
- **Where**: `lay_out_code` in `convert.rs` of `rm2k-snes/convert/`.
- **More**: [Events](../../internals/events.md); measured in
  [Frozen World's battle system](../../research/2026-10-15-frozen-worlds-battle-system.md).

## Copies of an event share a row

Events of a map whose pages are the same, field by field, that draw nothing
and never move —a strip of invisible touch exits along an edge, a grid of
the same sign— keep only the row of the one with the lowest id in the RAM's
event tables; the others are a cell in a ROM table behind the map's
positions, and the cell fires the first one's page. A map of 284 events
needs 38 rows.

- **Saves**: RAM —an event's row is bank 0's, and there are `EV_MAX`—; and
  it is what keeps a map over that cap from losing anything.
- **Not lossless in one case**: a twin on the hero's layer stops the hero,
  but not an event that walks into it.
- **Where**: `trim.rs` in `rm2k-snes/convert/`; `twin_in_cell` in
  `runtime/action_key.s` and `hero_blocked_rest` in `runtime/twins_block.s`.
- **More**: [Events that do not fit in the RAM](../../divergences/events-that-do-not-fit-in-the-ram.md).

## The pure parallel common events run only when what they read changes

A parallel common event that only reads switches and equipment, and only
writes skills and the three permissions, runs when its switch turns on and
on the frame after something it reads or writes changes, instead of every
frame. Between those moments it would give the same result.

- **Saves**: CPU —a body that took about an eighth of the frame—.
- **Where**: `pure` in `rm2k-snes/events/src/commons.rs`; `compar_tick` in
  `runtime/common_events.s`.
- **More**: [The pure parallel common events run only when what they read changes](../../divergences/pure-parallel-common-events-run-only-when-what-they-read-changes.md).

Since 2026-10-17 a map's parallel page follows the same rule when it is
pure the same way, or also sets the panorama —`ChangePBG`—: The Frozen
World's `BackdropSetter`, 113 `if switch → ChangePBG` on its battle screen,
cost ~110 of the frame's 262 lines.

- **Where**: `pure_page` in `rm2k-snes/events/src/commons.rs`, bit 7 of
  the page's trigger; `pure_due` in `runtime/event_pages.s`.

## The refresh runs at three moments

The active pages are recomputed when the interpreter lets go, at the
start of the frame, and the expensive half —OAM rows and route slots— on
the next frame; not before every command, as EasyRPG does, because no
opcode reads which page is active.

- **Saves**: CPU —it is what makes the frame fit in cutscenes that turn
  switches on by the bunch—.
- **Where**: `pick_pages` and `apply_pages` in `runtime/event_pages.s`.
- **More**: [The refresh runs at three moments](../../decisions/the-refresh-runs-at-three-moments.md).

## A refresh only when something changes, and only for what changed

A switch or a variable written with the value it already had asks for no
refresh: a variable is noted with the value it had, and the refresh is
asked only if, where it is looked at, it is not that value any more —a
parallel page that computes a percentage in place goes through two other
values every frame and comes back—. And a refresh sets up again only the
events whose active page changed, which is RM2k's `SetupFromPage`. And a
variable write asks for one only if some page condition of the map looks
at that variable: that is RM2k's `SetNeedRefreshForVarChange`, answered
by the event compiler as the write's `VAR_REFRESH` bit (`refresh` in
`rm2k-snes/events/src/compile.rs`).

- **Saves**: CPU —The Frozen World's Plate Placement refreshed its 425
  pages every frame and ran at six frames per tick; its battle screen at
  four—.
- **Where**: `set_switch`, `var_note` and `var_settle` in
  `runtime/variables.s`; `page_applied` in `runtime/event_pages.s`.
- **More**: the task
  [`frozen-world-the-battle-maps-lag`](https://github.com/lautarodragan/the-advocate-work/blob/main/tasks/frozen-world-the-battle-maps-lag.md).

## One dispatch table for the opcodes

`run_code` dispatches with one `jmp (table, x)`, built from the bytecode's
declaration, instead of a chain of comparisons.

- **Saves**: CPU on every opcode.
- **Where**: the `EVENT_HANDLERS` macro, from `rm2k-snes/bytecode/src/events.rs`.
- **More**: [Events](../../internals/events.md).

## The refresh leaves its answers for the frames that follow

The refresh is the only moment an event's active page changes, so what
it finds is kept until the next one: the list of the parallel pages, each
with its event and its page already resolved, and the active page of
every event, which a page that starts reads instead of walking the map's
pages again. The sweep that marks the autorun pages every frame does not
run when the refresh counted none.

- **Saves**: CPU on every frame —a parallel page used to be looked for
  and its header read from the cartridge again every frame—.
- **Where**: `record_parallel` and `apply_pages`, `one_parallel`,
  `page_from_pick` and `wait_autostart` in `runtime/event_pages.s`;
  `PAIR_LIST_BYTES` in `runtime/far_ram.s`.
- **More**: the comments of those routines.

## `OP_CELL_TEST`: four commands in one opcode

RM2k's way of asking whether two characters touch —two `ControlVars`
that copy a character's cell into variables and two nested
`ConditionalBranch` that compare them with two others— is compiled into
one opcode that does exactly the same, writes and refresh included.

- **Saves**: CPU —three turns of the interpreter per check, on pages that
  run every frame—.
- **Where**: `fadable_cells` in `rm2k-snes/events/src/compile.rs`;
  `op_cell_test` in `runtime/interpreter_variables.s`, and its body in
  `runtime/interpreter_cells.s`.
- **More**: the header of `runtime/interpreter_cells.s`.

## A parallel that stops at its cell test runs without its interpreter

The parallel pages that start with `OP_CELL_TEST`, are not halfway through
anything, do not move and fail the test end there, so they are run back to
back, the test and nothing else, without saving and restoring each one's
interpreter. The first one that is not like that goes on with its own.

- **Saves**: CPU —finding, saving and giving back each page's state,
  every frame—.
- **Where**: `parallel_cell` in `runtime/interpreter_cells.s`, from
  `parallel_tick` in `runtime/event_pages.s`.
- **More**: the comment on `parallel_cell`.

## The commands the parallel pages run every frame take the short way

In the interpreter, the cases the parallel pages repeat every frame go
first and short: a comparison of a variable with a constant for
"equal" is one sixteen-bit subtraction, and `ControlVars` looks for the
character's-cell operand before the others.

- **Saves**: CPU on every frame of those pages.
- **Where**: `op_if_var` in `runtime/interpreter_conditions.s`; `op_var`
  in `runtime/interpreter_variables.s`.
- **More**: the comments there.
