# MAD-Elevator

An endless runner that calculates high scores — vertically.

A glass elevator tears loose from its shaft and keeps going: up through the
low airspace, past the cranes, through the debris, across the flight levels
and out into orbit. You steer left and right. It never stops climbing. The
only number that matters is how many feet you made it before something hit
you.

Everything is one self-contained file — `index.html`. No build, no
dependencies, no assets: the art is drawn with the canvas 2D API at runtime.

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

| From | Band | What is up there |
| --- | --- | --- |
| 0 ft | Low Airspace | birds |
| 1,800 ft | Construction | crane booms and scaffolding — fly the gap |
| 4,500 ft | Debris Field | falling brick, concrete slabs, rusted I-beams |
| 9,000 ft | Flight Level | airliners crossing your climb |
| 16,000 ft | Stratosphere | traffic and tumbling wreckage, faster |
| 24,000 ft | Low Orbit | satellites, and saucers that follow you |
| 36,000 ft | Deep Space | all of it at once, including a crane |

Climb speed ramps from 360 to 1,150 px/s over the first 36,000 ft, and the
spawn interval tightens with it. Squeaking past a hazard scores a near miss
and a small altitude bonus. Your best climb is kept in `localStorage`.

## Layout of `index.html`

The script is sectioned in the order it runs: world constants, helpers,
canvas fitting, altitude bands, audio, state, input, screen glue, hazards,
particles, simulation, rendering, main loop, boot.

Tuning knobs worth knowing: `FEET_PER_PX` (how much altitude a pixel of
travel is worth), `SPEED_MIN` / `SPEED_MAX`, `RAMP_FEET` (how long the
difficulty takes to max out), and the `ZONES` table, which maps an altitude
to the mix of hazards that spawn there.
