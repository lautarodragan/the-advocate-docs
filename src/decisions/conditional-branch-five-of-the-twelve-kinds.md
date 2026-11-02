# ConditionalBranch: five of the twelve kinds

## What

**`ConditionalBranch`: five of the twelve kinds.** Switch, variable, gold,
item and actor, and of the actor kind four of its seven questions —is in
the party, has at least this many HP, knows the skill, has the item
equipped—. The HP one came on 2026-10-15, for The Book of Three's parallel
common event `Reraise KO'd chars`; of the three left, the level is asked 321
times by Vampires Dawn and the state 9, and the name by no game.

## Why

The criterion with which they were added, from when only the switch kind
existed, and it still holds: **modelling a branch that nothing exercises is
inventing a difference**. With the switch kind alone, 21 of the 24 pages
the command unlocked got in, and the three missing ones asked whether the
hero was in the party, which without a variable party always gave the same
answer.

## How it compiles

To two opcodes: `OP_IF` —switch, polarity and target— and `OP_GOTO`, the
unconditional jump that goes where the `else` was. **The target is absolute
within the page**, so in the ROM jumping is assigning the cursor; which
target that is gets resolved by the converter, the only one that sees the
whole page, by emitting it blank and patching it when the branch closes.

Nesting comes from a stack and not from looking at the indentation the way
`SkipToNextConditional` does: it walks a flat list, while here the blocks
already come nested.

**Kind 8, "started by the decision key", costs the ROM nothing**, since
2026-11-01: how a page was started is fixed by its trigger for most of
them, so the compiler answers it. easyrpg's `triggered_by_decision_key` is
true only on a frame pushed as an action: an action page always is, and
a parallel or automatic page, a common event or a called body never is.
A yes compiles to nothing and a no to the `OP_GOTO` past the branch; on a
touch or collision page, which starts either way, and on a troop page, it
does not compile. The seven that any ported game asks are Zounds' jail
guards, on parallel pages, where it is always no. The test is
`started_by_the_decision_key_is_answered_when_compiling` in
`rm2k-snes/events/src/compile.rs` in the-advocate.
