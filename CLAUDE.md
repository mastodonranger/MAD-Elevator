# MAD Elevator

A 9:16 portrait endless runner in a single self-contained `index.html`. A glass
elevator car flies from street level into space. 540×960 logical world,
letterboxed; `FEET_PER_PX = 0.07`.

## Art direction — read this before drawing anything

The target is **Sega Genesis sprite work**. This is not a vague mood; it is a
specific set of properties, and every new asset has to hit them. The house
sprites already do — measure against them rather than guessing.

### The numbers

Count the colours in a finished sprite and the share of opaque pixels the
commonest one takes. The existing set:

| sprite  | colours | dominant |
|---------|---------|----------|
| debris  | 5–6     | 41–69%   |
| pickup  | 6       | 34%      |
| bird    | 8–10    | 33%      |
| heli    | 16      | 17%      |
| car     | 31      | 17%      |

Small tumbling objects live at the top of that table: **4–6 colours, one of
them covering 40%+**. That lopsided split is the whole look. An even ramp —
31/26/18/15/10 across five tones — is a gradient wearing five colours and
reads as modern indie pixel art, not Genesis. If the histogram is flat, the
sprite is wrong no matter how it looks at 3×.

### The rules that produce those numbers

- **One flat body colour.** Then shading laid on as a few large contiguous
  patches: one shadow shape, one highlight shape. Not a per-row ramp.
- **Hard near-black outline** down both flanks, doubled where the form turns
  away hardest.
- **No dithering as shading.** Bayer fields read as noise beside flat regions.
  Dither only where a real Genesis game would: a transparency effect.
- **No per-pixel texture spray.** `speck` is an accent at ~1% of the sprite,
  not a surface treatment. If it is breaking up every seam, the seams are
  wrong.
- **Features are flat shapes.** A crater is a flat disc with one pixel of rim
  on the sunward side. Anything dished or bowled reads as an airbrush.
- **Silhouettes are hand-laid row tables** — `[[y, x0, x1], ...]` — not a noise
  function evaluated at runtime. See `debrislib.js` for the reference
  implementation. A generator cannot produce a deliberate outline.
- **Boundaries step in runs**, four to six rows, never jitter a pixel per row.
- **Light comes from the upper left.** Space objects get one hard source and a
  sharp terminator; city objects get the softer three-tone ramp.

### Matching the style means matching the technique, not transplanting features

The debris set's straight-edged details work because debris *is* straight-edged.
Ruled bars and flat stripes put onto a round rock read as paint. Take the
method — row tables, flat regions, hand placement — and let the forms follow
the object.

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
