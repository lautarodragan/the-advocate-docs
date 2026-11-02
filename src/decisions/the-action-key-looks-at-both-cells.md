# The action key looks at both cells

## What

**The action key looks at both cells, and each one asks for the opposite
layer.** `CheckEventTriggerHere` wants `GetLayer() != Layers_same` and
`CheckEventTriggerThere` wants `== Layers_same`.

## Why

It isn't symmetry for decoration: an event on the hero's layer blocks the
way, so the hero could never stand on top of it; one on another layer
doesn't stop them, and they walk over it.

The filter **lives in the ROM and not in the converter**, because the same
page counts or not depending on which cell is looking at it.

## How it applies

**Both cells fire, not the first that answers.** `CheckActionEvent` ORs
three passes —the touch pages in front, the action pages of the hero's
own cell, the action pages in front— and every event that queues runs, one
behind the other by its number. Until 2026-11-02 the ROM stopped at the
first pass that queued, and the first rock of Zounds, on map 33, could not
be mined from above: its top half is an action event with an empty page
over the hero, and its bottom half, in front, was never looked at
([task](https://github.com/lautarodragan/the-advocate-work/blob/main/tasks/the-action-key-stops-at-the-event-under-the-hero.md)).
