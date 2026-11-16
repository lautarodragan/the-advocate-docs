# A route's transparency is all or nothing

## What

A route's commands 40 and 41 raise and lower its character's transparency
by an eighth: RM2k keeps a level from 0 to 7 and draws the character at
opacity `(8 - level) * 32 - 1` —255, 223, 191, 159, 127, 95, 63 and 31 out
of 255—. It never reaches zero: seven 40s leave a ghost.

The ROM keeps the same level, on events and on the hero, and **draws the
character while the level is under 4 and hides it from 4 on**: half opaque
or more is drawn whole, less than half is not drawn. A page marked
translucent starts at 3, as in RM2k —it is drawn baked, see [Translucent
events end up opaque](translucent-events-end-up-opaque.md)—, so a single 40
hides it. See `ev_transparency` in `runtime/far_npcs.s` in the-advocate.

What is lost:

* **The fade.** Five 40s in a row are one frame in RM2k, so the steps are
  not seen there either; a 40 now and another later are, and here the
  character is there until the fourth and gone from it.
* **The ghost.** A character at level 4 to 7 is still seen in RM2k, faint;
  here it is not.
* **An event whose page draws nothing keeps no level**, and one whose
  graphic does not follow its page keeps its level when the page changes,
  where RM2k sets it again to the new page's.

Half drawn would be colour math on the object, which was tried for
translucent events and left out: see "Colour math for translucent events" in
[Tried and left out](../optimizations/tried-and-left-out.md).

## Measured

On 2026-11-16, over the eight ports' forced routes —no page's own route has
a 40 or a 41—: in A Blurred Line, only the computer screen of map 2, four
events that five 40s each fade into the starry panorama when Core gets in
and five 41s bring back; RM2k leaves them at 95 out of 255, and the stars
are seen with the screen's grid faint on top. El Dorado has the most, about
350 `MoveEvent`s of a single 40 on cars and obstacles; Vampires Dawn,
Sunset over Imdahl and Zounds fade lights, a few events and the hero, out
and back in. Until
that day 40 and 41 were dropped, and the screen stayed dark.

## When it stops being enough

A fade that is the point of a scene, slow enough to be seen step by step, or
a character meant to be seen faint.
