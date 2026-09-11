# MAD-Elevator

An endless runner that calculates high scores — vertically, drawn as
monochrome pixel art in the vein of Canabalt.

A glass elevator tears loose from its shaft and keeps going: up through the
low airspace, past the cranes, through the debris, across the flight levels
and out into orbit. You steer left and right. It never stops climbing. The
only number that matters is how many feet you made it before something hit
you.

Everything is one self-contained file — `index.html`. No build and no
assets: the art is drawn with the canvas 2D API at runtime. The only
external request is the Google Fonts stylesheet for the two pixel
typefaces; offline it falls back to the system monospace stack and plays
exactly the same.

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
| touch and drag | the car tracks your finger |
| `Space` / `Enter` / tap | launch, and ride again after a crash |
| `Esc` / `P` | hold |
| `R` | restart mid-run |
| `M` | mute |

## Altitude bands

Hazards are picked by how high you are, and the sky changes with you — day
haze, deep blue, star field, then the curve of the earth sinking away below.

| From | Band | What is up there | Reached at |
| --- | --- | --- | --- |
| 0 ft | City Level | birds, and the skyline you are leaving | — |
| 3,000 ft | High Rise Construction | stationary crane booms with a gap to fly, and swinging jibs with a wrecking ball on a cable | 0:19 |
| 8,000 ft | Debris Field | more cranes, and falling brick | 0:39 |
| 14,000 ft | Flight Level | airliners crossing your climb | 0:56 |
| 22,000 ft | Stratosphere | traffic and tumbling wreckage, faster | 1:11 |
| 32,000 ft | Low Orbit | satellites, and saucers that follow you | 1:29 |
| 44,000 ft | Deep Space | all of it at once, including a crane | 1:47 |

("Reached at" is a clean run with no crashes — the climb is deliberately a
long one, and the first band is a slow look at the city.)

Difficulty is anchored to the bands rather than a flat curve: each one is a
step up in climb speed (150 → 900 px/s, about 107 to 460 mph) and in spawn
pressure (one hazard every ~2.4s at street level, every ~0.45s in deep
space), ramping across its own height.

Squeaking past a hazard scores a near miss and a small altitude bonus. Your
best climb is kept in `localStorage`.

## Power-ups

Both pulse red — the only colour in the game, so it always means "this helps
you" — and both are drawn with a hard ink outline, so they never rely on
colour alone to be seen. While one is running, a label sits at the top of
the screen; the labels stack, and you can hold both at once.

**VMS3** — a bubble, first found just inside Flight Level at 14,200 ft, then
at a random height every 7,000–13,000 ft after. It absorbs one crash: the
hit pops the bubble instead of ending the run, and you get 1.3 seconds of
mercy to fly clear of whatever you hit. Labelled `VMS3 ACTIVATED`.

**F1 System Speed** — a rocket, from 17,000 ft, then every 8,000–15,000 ft.
It lifts the car a literal 200 ft up the screen and runs the world 2.6×
faster for three seconds, and nothing can touch you while it burns. The
surge comes in fast, holds, then eases off while the car settles back down
to its station — the wind-down is the part that tells you it is ending.
Labelled `F1 SYSTEM SPEED`.

## The look

The game renders into a canvas that is **180×320 device pixels**. Everything
is authored in 540×960 world units and the context is scaled by `1/PIX`, so
the rasteriser itself does the pixelating and CSS blows the result back up
with nearest-neighbour sampling. Sprites are built from rects on that grid;
debris and satellites quantise their spin to four 90° frames so nothing ever
lands off-grid.

There is one palette — a desaturated blue-grey ramp — and no second hue
anywhere except the red the power-ups own. Every colour is derived from the current sky by value: the three
skyline layers mix toward ink by 0.20 / 0.44 / 0.68, so depth reads as
contrast. Once the sky is dark enough to swallow a plain silhouette,
hazards pick up a one-pixel backlit rim. The exhaust plume is the only
bright thing in the world, with the crane gap lamps allowed to borrow it,
because that is what you aim at.

Contrast is a fairness requirement, not a finish: hazards are ink on a pale
sky and gain a two-pixel backlit rim once the sky is dark enough to swallow
them; power-ups and gap lamps are drawn with an ink outline so they read on
any ground; background scaffolding is painted in a mid value so it can never
be mistaken for something that will kill you; and the HUD flips its ink
*and* its backing together with the sky, so the readout holds the same
contrast at 500 ft and at 50,000.

## Layout of `index.html`

The script is sectioned in the order it runs: world constants, helpers,
canvas fitting, altitude bands, audio, state, input, screen glue, hazards,
particles, simulation, rendering, main loop, boot.

Tuning knobs worth knowing: `FEET_PER_PX` (how much altitude a pixel of
travel is worth), `SPEED_MIN` / `SPEED_MAX`, `BAND_TAIL` (how far the last
band keeps ramping), `SHIELD_FIRST` / `SHIELD_INV`, `ROCKET_FIRST` /
`BOOST_TIME` / `BOOST_RISE` / `BOOST_MULT`, `PIX` (the size of a pixel), and the `ZONES` table, which maps an altitude to the mix of hazards
that spawn there and drives the difficulty curve through `difficultyAt()`.
