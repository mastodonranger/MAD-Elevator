# MAD Elevator

A 9:16 portrait endless runner in a single self-contained `index.html`. A glass
elevator car flies from street level into space. 540×960 logical world,
letterboxed; `FEET_PER_PX = 0.07`.

## Working on assets — the process

**Design and confirm every asset first. Do not wire anything into the game
until the whole set is signed off.** Asset work is the primary track unless
told otherwise.

Present each asset as a **sheet**, in the format of `birds.html` and the plane
sheet: a large pixel grid at 6-7x zoom with a faint cell grid behind it,
numbered candidates down the left with a name and a one-line note on what makes
each distinct, animation frames or tumble angles across the columns, and a
footer stating the cell size, the facing, and the technical constraint the art
is working to.

Show candidates, take the picks, iterate on the picks. Only once the whole set
is agreed does any of it go near `index.html`.

Once a design and its animation are confirmed, they go into the **motion tests
doc** — <https://claude.ai/artifact/VVuUantPqsLPbzjYmGUS7w> — and that doc is
what gets presented. It already holds the elevator tilt, birds, crane, debris,
helicopters, aircraft and power-ups. Update it; do not start a second viewer.

## Art direction — read this before drawing anything

The reference is the **planes, cranes and elevator car**, not the debris. Those
are the sprites the game is judged by, and they are *crisp*: bright saturated
fills, edges that step on a coarse grid, a light line along the top, and detail
made of distinct **parts** — window rows, a lattice, a painted stripe — never
surface texture.

### What the numbers actually mean

| sprite  | colours | dominant |
|---------|---------|----------|
| car     | 31      | 17%      |
| heli    | 16      | 17%      |
| plane   | 15      | 17%      |
| bird    | 8–10    | 33%      |
| pickup  | 6       | 34%      |
| debris  | 5–6     | 41–69%   |

A plane carries 15 colours because it has a fuselage, a window row, a tail, an
engine and a stripe — each flat filled with two or three tones. **Not** because
one shape is shaded richly. Colour count follows part count. Do not chase the
number; chase the parts.

### The rules

- **Flat fills, hard edges.** Two or three tones per part, no ramps.
- **No texture, anywhere.** No dithering as shading, no per-pixel grit spray.
  There is not a single noisy pixel in the reference set. If a surface needs
  interest, give it a *thing* — a crater, a seam, a panel line — not noise.
- **Silhouettes step on a 3px grid.** Long constant runs with deliberate steps
  is what gives the boom and the heli their clean edges. Per-row jitter reads
  as ragged.
- **A light line along the top edge**, two pixels, following the silhouette
  down the flanks and stopping where the form turns away. Follow the edge
  column by column — filling the top third of the rows just bleaches it.
- **Hard near-black outline** down both flanks.
- **Features are their own parts.** A crater is a flat floor, a hard outline
  and a bright rim on the sunward arc — built like a cockpit window, not
  shaded like a dent. Anything dished or bowled reads as an airbrush.
- **One colour beat per sprite** where it earns it — the red stripe on the news
  helicopter, a seam of exposed ice on a rock. One, not three.
- **Silhouettes are hand-laid row tables** — `[[y, x0, x1], ...]` — not a noise
  function evaluated at runtime. See `debrislib.js` for the format.
- **Light comes from the upper left.**

### Matching the style means matching the technique, not transplanting features

The debris set's straight-edged details work because debris *is* straight-edged.
Ruled bars and flat stripes put onto a round rock read as paint. Take the
method and let the forms follow the object.

## Pipeline

Sprites are **baked once at 1:1 into offscreen canvases at boot, then blitted**
(`SPR.bake()`), never drawn live per frame. `put()` blits with
`imageSmoothingEnabled = false` and supports rotation, which is how the
tumbling families work.

- Bake at 1:1. **Never scale a sprite** — resampling is the one thing that
  gives the pixel grid away. A continuous size range becomes discrete buckets.
- **Hitboxes come from the baked alpha**, not from numbers typed next to the
  spawn. The sprite is the hitbox.
- The bake checks every surface for edge-touching and reports `clipped`; a
  non-zero count means a surface was sized too small.

## Reference-driven work

Landmarks and real objects are **traced, not eyeballed**. The CN Tower was
measured off a photograph: the vertical scale pinned from tip and ground, every
width read off a row-by-row silhouette trace against clean sky, two traces run
from opposite directions with the narrower taken per row so a cloud touching the
tower cannot widen it. Report error in pixels against the reference, and say
which parts are extrapolated rather than measured.

## Conventions

- Develop and push only to the branch named in the session prompt.
- Never put model identifiers in commit messages, code comments, or anything
  else pushed to the repo.
- Do not create pull requests unless asked.
- Verify in the running game, not just in a test harness. Measure bake cost and
  frame time; both are reported by the debug hooks (`?debug`, `DBG.*`).
