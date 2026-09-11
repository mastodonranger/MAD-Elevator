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

| From | Band | What is up there |
| --- | --- | --- |
| 0 ft | Low Airspace | birds |
| 1,800 ft | Construction | crane booms and scaffolding — fly the gap |
| 4,500 ft | Debris Field | falling brick, concrete slabs, rusted I-beams |
| 9,000 ft | Flight Level | airliners crossing your climb |
| 16,000 ft | Stratosphere | traffic and tumbling wreckage, faster |
| 24,000 ft | Low Orbit | satellites, and saucers that follow you |
| 36,000 ft | Deep Space | all of it at once, including a crane |

Difficulty is anchored to the bands rather than a flat curve: each one is a
step up in climb speed (210 → 1,080 px/s) and in spawn pressure (one hazard
every ~1.9s at the ground, every ~0.4s in deep space), ramping across its
own height. The opening band is deliberately slow and sparse.

Squeaking past a hazard scores a near miss and a small altitude bonus. Your
best climb is kept in `localStorage`.

## The bubble

A shield bubble waits just inside Flight Level, at 9,200 ft, and after that
one appears at a random height every 6,500–12,000 ft. Fly into it and the
car carries a bubble that absorbs one crash: the hit pops the bubble instead
of ending the run, and you get 1.3 seconds of mercy to fly clear of whatever
you hit. `SHIELD UP` in the HUD means you have one banked.

## The look

The game renders into a canvas that is **180×320 device pixels**. Everything
is authored in 540×960 world units and the context is scaled by `1/PIX`, so
the rasteriser itself does the pixelating and CSS blows the result back up
with nearest-neighbour sampling. Sprites are built from rects on that grid;
debris and satellites quantise their spin to four 90° frames so nothing ever
lands off-grid.

There is one palette — a desaturated blue-grey ramp — and no second hue
anywhere. Every colour is derived from the current sky by value: the three
skyline layers mix toward ink by 0.20 / 0.44 / 0.68, so depth reads as
contrast. Once the sky is dark enough to swallow a plain silhouette,
hazards pick up a one-pixel backlit rim. The exhaust plume is the only
bright thing in the world, with the crane gap lamps and the bubble allowed
to borrow it, because those are the two things you need to see.

## Layout of `index.html`

The script is sectioned in the order it runs: world constants, helpers,
canvas fitting, altitude bands, audio, state, input, screen glue, hazards,
particles, simulation, rendering, main loop, boot.

Tuning knobs worth knowing: `FEET_PER_PX` (how much altitude a pixel of
travel is worth), `SPEED_MIN` / `SPEED_MAX`, `BAND_TAIL` (how far the last
band keeps ramping), `SHIELD_FIRST` / `SHIELD_INV`, `PIX` (the size of a
pixel), and the `ZONES` table, which maps an altitude to the mix of hazards
that spawn there and drives the difficulty curve through `difficultyAt()`.
