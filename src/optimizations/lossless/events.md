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

## Every jump lands where the compiler already said

A `ConditionalBranch` that fails, the `else` that skips the other branch,
a choice's options, a battle's handlers, `JumpToLabel`, `EndLoop` and
`BreakLoop` all carry their destination as an index within the page, which
the compiler resolved because it is the only one that sees the whole page:
on the console, jumping is assigning the cursor. RM2k finds the same place
by walking the list of commands —`SkipToNextConditional`,
`CommandJumpToLabel`, `CommandEndLoop`—, looking at indentations or at
label numbers.

- **Saves**: CPU on every branch that does not hold, every frame on the
  parallel pages; and since the index is relative to the page, a page can
  be moved —`CODE_CARRY`— or shared —[identical pages are stored
  once](#identical-pages-are-stored-once)— without touching it.
- **Where**: `patch`, `label_targets`, `loop_start` and `break_target` in
  `rm2k-snes/events/src/compile.rs`; `op_if`, `op_if_var` and `op_goto` in
  `runtime/interpreter_conditions.s`, `op_choice` in
  `runtime/interpreter_message.s`, `op_battle_destination` in
  `runtime/interpreter_shop_and_battle.s`.
- **More**: [ConditionalBranch: five of the twelve kinds](../../decisions/conditional-branch-five-of-the-twelve-kinds.md#how-it-compiles).

## A command that waits goes back to itself

A command that cannot go on yet —a `KeyInputProc` that waits and finds no
key, a parallel's message while another one is on screen, "wait for the
move routes" with one still moving, a `GameOver` behind someone's message,
a level-up message waiting for the window— puts the cursor back on its own
opcode and ends the frame; the next frame runs it again. Nothing says "I am
waiting" anywhere.

- **Saves**: RAM —a parallel page's state is seven bytes, and a waiting
  flag per command and per interpreter would be more—, and the code that
  would resume from it.
- **Where**: `op_key` in `runtime/interpreter_keys_and_waits.s`;
  `op_message` and `op_number` in `runtime/interpreter_message.s`;
  `op_wait_routes` in `runtime/interpreter_movement.s`; `op_game_over` in
  `runtime/interpreter_game_over.s`; `op_level_page` in
  `runtime/interpreter_party.s`.
- **More**: the comment on `op_key`.

## The parallel common events save the foreground only when one runs

Every frame each parallel common event looks at its switch, and the ones
that do not run —almost all, almost always— cost that and nothing else:
the foreground interpreter's state is saved in far RAM by the first one
that does run, and given back once after the last.

- **Saves**: CPU on every frame, on every map.
- **Where**: `compar_tick` and `compar_one` in `runtime/common_events.s`.
- **More**: the comment on `compar_tick`.

## Only the variables the game names get a slot

RM2k declares thousands of variables; the compiler sweeps every page,
common event and troop page of the game and gives a slot, from zero up,
only to the variables some command or condition names, and the bytecode
names a variable by its slot. The switches are a bitmap only as long as the
highest switch the game names.

- **Saves**: far RAM and SRAM —two bytes per variable the game uses, not
  per variable it declares, and the same in the saved game—, and a
  variable's operand is one byte. The slots come from the whole game, not
  from a cartridge's maps, so a saved game loads in any cartridge of it.
- **Where**: `variables` and `switch_top` in
  `rm2k-snes/events/src/indices.rs`; `var_read` and `var_write` in
  `runtime/variables.s`; `SWITCH_BYTES` in `runtime/switches.s`.
- **More**: the doc comment on `variables` in `indices.rs`; the cap on
  slots is [lossy](../lossy/tables-of-a-fixed-size.md#variables).

## A skill learned is one bit, for the pairs the game can ask about

Which skills each actor knows is world state that travels in the save. It
is kept as one bit per (actor, skill) pair, and only for the pairs the game
can look at: the ones the actors learn by level, and the ones some
`ChangeSkills` or "knows the skill" branch names. The compiler puts the
pair's bit where the command carries its operand, so the console does not
need the actor to find it.

- **Saves**: far RAM and SRAM —no list per actor, with its searching,
  sorted insertion and cap—.
- **Where**: `skills` in `rm2k-snes/events/src/indices.rs`; the
  `FAR_SKILL` bits, read by `skill_known` in
  `runtime/equipment_from_events.s` for `op_if_actor` in
  `runtime/interpreter_conditions.s`.
- **More**: the doc comment on `skills` in `indices.rs`.
