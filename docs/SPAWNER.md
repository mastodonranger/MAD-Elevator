# Spawner spec

How hazards are placed, and the rule that keeps a run winnable.

This replaces the current per-type spawners (`spawnCrane`, `spawnDebris`,
`spawnBird`, …) with a single placement pass that will not commit a hazard
until it has checked that a path still exists.

---

## 1. The invariant

> At every point in a run there is at least one lateral position the car can
> reach in the time available that is clear of every hazard.

A run that violates this is **unwinnable from that moment on**, no matter how
well the player is playing. That is the failure we are ruling out. We are not
promising the run is comfortable, or that a human will find the path — only
that one exists.

---

## 2. The reachability model

Work in **car-centre space**: the set of x positions the car's centre could
occupy, represented as a sorted, disjoint list of intervals over
`[HALF, W - HALF]` where `HALF = PLAYER_W / 2 = 42`.

Two operations advance the state.

**Dilate** — between one hazard and the next, the car can move. Over a climb
distance `dp` at speed `v` it has `t = dp / v` seconds, and covers

```
travel(t) = t <= VMAX/A  ?  0.5 * A * t²
                         :  0.5 * A * (VMAX/A)² + (t - VMAX/A) * VMAX
```

with `A = PLAYER_ACCEL = 2600` and `VMAX = PLAYER_MAXV = 640`. Every interval
grows by `travel(t)` on both sides, clipped to the play area.

**Subtract** — a hazard removes its blocked span from the set.

If the set is ever empty, the run is dead.

### Blocked spans

A hazard's span is widened by `HALF`, because we are tracking the car's centre.
Movers are widened again by however far they drift while the car is passing
them.

| Hazard | Blocked span (car-centre space) |
| --- | --- |
| Crane, left | `[0, reach + HALF]` |
| Crane, right | `[W - reach - HALF, W]` |
| Debris | `[x - half - HALF - d, x + half + HALF + d]` |
| Bird | same as debris |

where `d = drift * (band / v)` — lateral speed times the time the car spends
inside the hazard's vertical band.

### Implementation trap

Dilation makes neighbouring intervals **overlap**. If they are not merged back
into a disjoint set before the next subtraction, one subtraction can split
several overlapping intervals at once and the count doubles per hazard — 128
intervals by the seventh, and the check hangs. Merge after every dilate. With
the merge in place the set never exceeded **2 intervals** across 180,000
hazards.

---

## 3. Placement

One hazard at a time. Roll its type, shape and the gap since the last one,
then **test before committing**:

```
candidate = subtract(dilate(state, travel(gap / v)), blockOf(hazard))

if width(candidate) >= CORRIDOR_MIN:
    commit
else:
    adjust and retry (up to 10 times)
```

Adjustments, in order of preference:

1. **Widen the gap** — `gap *= 1 + attempt * 0.22`. Costs nothing but pacing.
2. **Pull a crane's reach back** — `reach -= 34`, floor 120.
3. **Slide a mover** — move a bird or debris toward the middle of the widest
   surviving interval.
4. **Drop the hazard** — last resort. In 180,000 placements this never fired.

`CORRIDOR_MIN = 40` px of car-centre freedom. Positive width alone means the
car fits exactly; 40px means it does not have to be frame-perfect.

### Crane side changes

A side change has a cost the player must pay in lateral movement. The car only
has to **clear the tip**, not reach the middle of the passage, so for
consecutive reaches `R1` and `R2`:

```
cross = max(0, R1 + R2 - (W - 2 * (HALF + MARGIN)))     MARGIN = 12
gap  >= v * travelTime(cross) * 1.12                    12% slack
```

Worked examples at 620 px/s:

| R1 + R2 | Centre-to-centre | Actually required | Gap needed |
| --- | --- | --- | --- |
| 380 + 380 | 380 px | 328 px | 394 px |
| 380 + 320 | 350 px | 268 px | 336 px |
| 320 + 320 | 320 px | 208 px | 278 px |

Centre-to-centre over-charges by roughly 15–35%, which is why the earlier
"444px" figure was wrong.

### Same-side runs

At full difficulty the car can move **128 px** between hazards while a switch
costs **328 px**. Alternating every boom is therefore impossible up there, not
merely hard. Cap same-side runs at 3 and let the switch gap absorb the cost;
same-side pairs are free to dodge and buy back exactly the room the crossing
needs.

---

## 4. Why tuning is not enough

Measured over 3,000 generated runs spanning the full difficulty range:

| Guards | Forced game-overs |
| --- | --- |
| None | 944 / 3000 (31%) |
| + switch spacing | 324 / 3000 |
| + crane/other separation 150 px | 324 / 3000 |
| + reach capped at 390 | 272 / 3000 |
| + reach capped at 360 | 153 / 3000 |
| + separation 200 px | 133 / 3000 (4.4%) |
| **+ corridor check, floor 24 px** | **0 / 3000** |
| **+ corridor check, floor 40 px** | **0 / 3000** |
| **+ corridor check, floor 60 px** | **0 / 3000** |

Every parameter change helped and none made it safe. At 4.4% roughly one
player in twenty gets an unwinnable run and has no way of knowing it was not
their fault.

The tightest corridor observed equals the floor exactly in each case, which is
the signal that the guarantee is tight rather than accidental.

---

## 5. Cost

At `CORRIDOR_MIN = 40`, over 180,000 placements:

| | |
| --- | --- |
| Hazards dropped | 0 (0.00%) |
| Cranes pulled back a notch | 3,265 (1.81%) |
| Movers slid to a wider lane | 86 (0.05%) |
| Time per spawn | **0.6 µs** |

Against a 16,700 µs frame budget. The check is free.

---

## 6. Crane geometry

The crane is a **single boom reaching in from one edge**, not a full-width wall
with a hole. Difficulty is how far it reaches, so there is exactly one edge for
the player to judge.

| hazDiff | Max reach | Passage | Clearance over an 84px car |
| --- | --- | --- | --- |
| 0.0 | 236 | 304 px | 220 px |
| 0.5 | 342 | 198 px | 114 px |
| 1.0 | 420 | 120 px | 36 px |

36px of clearance at full difficulty is survivable but unforgiving, and that is
before a bird or a brick arrives in the same window. The corridor check makes it
*safe*; capping reach nearer 390 would make it *fair*. That is a design call,
not a correctness one.

Two booms can never share an altitude — each spawn produces one — and the
minimum gap (170 px at full difficulty) comfortably exceeds the 46 px beam, so a
sealed row cannot occur.

---

## 7. Known limits

- Hazards are modelled as a blocked x-span over a y-band. A mover that reverses
  direction mid-pass is approximated by widening its span by its full drift,
  which is conservative but not exact.
- The model assumes optimal play. It proves a path exists, not that a human
  finds it.
- Powerups are not modelled. Shield and VMS3 only ever make a run more
  survivable, so ignoring them keeps the check conservative.

---

## 8. Migration notes

Changes needed in `index.html` beyond the placement pass itself:

- `spawnCrane` becomes one-sided: `{ side, reach }` in place of
  `{ gapX, gapW, towerRight }`. The vertical mast is dropped.
- `rectsOf` for `crane` returns one rect, not two.
- The fixed `lastGapX ± 250` reachability clamp goes away — it is too tight at
  launch, where the car can cross 1,068 px, and far too loose at top speed,
  where it can manage 128. The corridor check replaces it.
- Debris kinds `slab` and `beam` become `rubble` and `cinder`; `CRASH_LINES`
  keys `debris:slab` and `debris:beam` need renaming and new copy.
- `BIRD_TONES` becomes three named species (pigeon, crow, sparrow) with
  per-species flap rates.
