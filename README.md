# MAD-Elevator

A vertical endless runner in a 9:16 frame. Car 3 has left the building and
it is not coming back down. You steer left and right; the only number that
matters is how many feet you made before something hit you.

Everything is one self-contained file — `index.html`. No build, no assets,
no network requests: the art is drawn with the canvas 2D API at runtime and
the type is the system sans stack.

## Play

Open `index.html` in a browser, or serve the folder:

```sh
python3 -m http.server 8000   # then visit http://localhost:8000
```

The playfield is locked to a 9:16 portrait frame and letterboxed into
whatever window it gets, so it plays the same on a phone and on a desktop.

## Controls

| Input | Action |
| --- | --- |
| `←` `→` or `A` / `D` | steer the car |
| hold left / right half | steers that way, like holding an arrow key |
| `Space` / `Enter` / tap | launch, and ride again after a crash |
| `Esc` | back to the menu from the results |
| `R` | restart mid-run |
| `M` | mute |

Touch is **directional, not positional**: the car goes the way of whichever
half of the screen you are holding and keeps going while you hold it, rather
than flying to your finger. Steering also tips the car into the turn, by
about six degrees in six quantised steps, so it leans without shimmering.

## The climb

Five zones. Difficulty is anchored to them, not to the clock: speed and the
spawn interval both interpolate across the zone you are in, so the ride
steps up as you cross a boundary rather than creeping.

| From | Zone | What is up there |
| --- | --- | --- |
| 0 ft | Shaft | gulls, while the city canyon climbs past you |
| 1,600 ft | Open Air | barriers with one gap, swinging wrecking balls, gulls |
| 4,000 ft | Traffic | aircraft crossing the lane, barriers, the odd gull |
| 8,000 ft | Ascent | rock, aircraft, tumbling debris, cloud decks |
| 14,000 ft | Vacuum | rock, debris and the occasional comet, under stars |

Climb speed runs from 150 to 820 px/s and the spawn interval tightens from
about one hazard every 2.3s to one every 0.46s. A pixel of travel is worth
0.30 ft, which is what keeps the altimeter readable rather than a blur.
Barriers and wrecking balls are rate-limited against each other so two
lane-blockers never stack, and a wrecking ball is always anchored near one
wall with its swing capped, so the far side of the shaft always keeps a lane
wide enough to fly through. Your best climb is kept in `localStorage`.

## Power-ups

Both are red, because red in this game only ever means *this helps you*.
While one is running its label sits at the top of the screen; the labels
stack, and you can hold both at once.

**VMS3** — a bubble, first at 3,200 ft and then every 3,200–5,600 ft. It
absorbs one crash: the hit pops the bubble instead of ending the run and
you get 1.2 seconds of mercy to fly clear. While it is up, the car is a
white silhouette inside a solid red bubble.

**F1 System Speed** — a rocket, first at 4,400 ft and then every
3,600–6,400 ft. It lifts the car 190px up the screen and runs the world
2.4× faster for 2.6 seconds, untouchable, then eases back down to station —
the wind-down is the part that tells you it is ending.

## The look

Monochrome and cel shaded. Every shape in the game is built the same way:
a flat base tone, **one** hard-edged shadow, and a heavy near-black
outline. No gradients, no dithering, no texture. `cel()` does it for an
arbitrary path (the shadow is a straight cut across the lower half, as if
the light came from the upper left), `celDisc()` for anything round (a hard
terminator), and `celRect()` / `celBar()` for the two rectangle cases.

**Only the sky interpolates.** `SKY` is a six-stop ramp from paper white at
street level to black in deep space, and it is the single authored colour
in the game. Everything drawn on top of it holds a fixed contrast so that
nothing can ever sink into the background at a crossover:

- **Buildings** are the sky pulled toward the ink, so they are always
  darker than it and can never match it. They exist only in the light half
  of the climb.
- **Hazards** are a constant bright face (`#ECECEC`) with a constant
  near-black shadow (`#232323`). Half of every hazard contrasts with a white
  sky and the other half with a black one, which makes readability a
  property of the art rather than a thing that has to be tuned per zone.
- **Clouds and worlds** are fixed tones, and background worlds deliberately
  sit in dim greys with a soft outline so they read as distance and are never
  mistaken for a rock you could hit.
- **Red** is reserved for exactly three things: the car, the two power-ups,
  and the posts marking the gap in a barrier.

The HUD's ink flips hard between near-black and near-white on the sky's
luminance, so the readout always takes the higher-contrast option.

**Two parallax layers of scenery**, at `0.16` and `0.38`. Low down they
stock with buildings, towers and tower cranes — flat grey slabs with a
shadow band down one side, black window grids on the near layer and a
coarser grid on the far one. Higher up they stock with cloud banks, and
higher still with worlds; a procedural starfield fades in from 9,200 ft.
Gaps are sized in each layer's own scrolled pixels, so roughly
`(screen + prop height) / gap` props are on screen at a time regardless of
how fast you are going.

## Crashing

The run does not simply stop. The frame takes a short **jiggle**, then
**freezes** with the debris hanging in mid-air. A red curtain wipes down
over the held frame carrying **GAME OVER**, holds for a beat, and fades to
black — and the results come up on that black: the feet travelled, what you
hit, whether it was a new best, and buttons for riding again or going back
to the menu.

## Layout of `index.html`

The script is sectioned in the order it runs: constants, helpers, palette,
cel-shading helpers, zones, canvas, audio, state, input, run control, the
world, simulation, scenery, rendering, main loop, boot.

Tuning knobs worth knowing: `FT_PER_PX` (how much altitude a pixel of travel
is worth), `SPEED_MIN` / `SPEED_MAX`, `ZONE_TAIL` (how far the last zone
keeps ramping), `SHIELD_AT` / `BOOST_AT` / `BOOST_*` / `MERCY`, the crash
beats (`CRASH_T`, `JIG_T`, `CURTAIN_AT`, `BLACK_AT`), `SKY` and the fixed
tone constants, `OUTLINE`, `PARA` and `scenStep()` for scenery density, and
the `ZONES` table, which maps an altitude to the hazards that spawn there and
drives the curve through `difficulty()`.
