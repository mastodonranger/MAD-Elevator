# MAD-Elevator

An endless runner that calculates high scores — vertically, drawn as
16-bit pixel art in one desaturated blue-grey ramp.

Car 3 is stuck in a parking garage. Maintenance has a different idea: the
car climbs its shaft, punches through the street slab, lights its rockets
and keeps going — up past townhouses and cranes and airliners, through a
meteor shower, past the moon and Mars and out into deep space. You steer
left and right. It never stops climbing. The only number that matters is
how many feet you made before something hit you.

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
| `Esc` / `P` | hold (on the results screen, back to menu) |
| `R` | restart mid-run |
| `M` | mute |

## The climb

Nine zones. Several run two phases internally, and the whole thing is
deliberately a long ride — the first zone alone is about half a minute of
flying up a street canyon.

| From | Zone | What is up there | Reached at |
| --- | --- | --- | --- |
| 0 ft | 1 · City | pigeons, while the buildings grow from townhouses to low rises, apartments, then high rises | — |
| 4,000 ft | 2 · Construction | moving crane booms, swinging jibs with a wrecking ball, falling girders and pallets — past corporate offices and skyscrapers | 0:27 |
| 9,000 ft | 3 · Skyline | a reprieve: sparse birds, and the CN Tower, One World Trade and the Burj Khalifa going by in the distance. Both power-ups are handed to you here | 0:50 |
| 14,000 ft | 4 · Low Airspace | helicopters and light aircraft, then jets from 18,000 ft | 1:07 |
| 22,000 ft | 5 · Upper Atmosphere | a meteor shower against a darkening sky; from 28,000 ft the lights go out entirely and space debris joins in | 1:29 |
| 34,000 ft | 6 · Orbit | debris and satellites, and the moon slides past | 1:57 |
| 46,000 ft | 7 · Asteroid Belt | asteroids, and a starfield that twinkles | 2:21 |
| 58,000 ft | 8 · Interplanetary | asteroids and alien saucers that follow you. Mars goes by | 2:42 |
| 72,000 ft | 9 · Deep Space | asteroids, saucers, the occasional comet — with ringed planets, nebulae and Voyager 1 in the black | 3:04 |

("Reached at" assumes a clean run.) Climb speed steps up zone by zone from
150 to 900 px/s — about 107 to 460 mph — and the spawn interval tightens
with it, from roughly one hazard every 2.4s at street level to one every
0.45s out past Mars. Squeaking past something scores a near miss and a
small altitude bonus. Your best climb is kept in `localStorage`.

## Power-ups

Both pulse red — the only colour in the game, so red always means "this
helps you" — and both carry a hard outline in the normal asset tone, so
they never rely on colour alone to be seen. While one is running a label
sits at the top of the screen; the labels stack, and you can hold both.

**VMS3** — a bubble, handed to you at 10,200 ft in the Skyline zone and
appearing at random every 11,000–19,000 ft after that. It absorbs one
crash: the hit pops the bubble instead of ending the run, and you get 1.3
seconds of mercy to fly clear. Labelled `VMS3 ACTIVATED`.

**F1 System Speed** — a rocket, first at 12,300 ft, then every
12,000–21,000 ft. It lifts the car a literal 200 ft up the screen and runs
the world 2.6× faster for three seconds, untouchable, then eases it back
down to station — the wind-down is the part that tells you it is ending.
Labelled `F1 SYSTEM SPEED`.

## The look

The game renders into a canvas that is **270×480 device pixels**. Everything
is authored in 540×960 world units and the context is scaled by `1/PIX`, so
the rasteriser itself does the pixelating and CSS blows the result back up
with nearest-neighbour sampling. Sprites are built from rects on that grid
in three tones — base, highlight, shadow — and anything that spins snaps to
quarter turns so nothing ever lands off-grid.

There is one palette: a desaturated blue-grey ramp, plus red for power-ups.
Everything else is derived from the sky by value — the parallax layers, the
window lights, the clouds.

**How an asset picks its colour** is the one rule worth knowing. It reads
the brightness of the sky *at its own height on the screen* and takes the
opposite: dark-on-bright below, light-on-dark above, always with an outline
in the other set. Zone 5's darkening lid then does what you would want
without a special case — hazards enter the frame white against the black
top and turn dark as they fall into the lit bottom half — and the full
inversion of deep space falls out of the same rule. Contrast is a fairness
requirement here, not a finish: it is what keeps the game playable at
50,000 ft and legible to a player who cannot rely on hue.

## Layout of `index.html`

The script is sectioned in the order it runs: constants, helpers, canvas,
zones, palette, audio, best, state, input, screens, scenery, hazards,
particles, simulation, rendering, main loop, boot.

Tuning knobs worth knowing: `FEET_PER_PX` (how much altitude a pixel of
travel is worth), `SPEED_MIN` / `SPEED_MAX`, `ZONE_TAIL` (how far the last
zone keeps ramping), `SHIELD_AT` / `ROCKET_AT` / `BOOST_*`, `PIX` (the size
of a pixel), the intro and crash beats (`I_*` and `X_*`), and the `ZONES`
table, which maps an altitude to the hazards that spawn there and drives
the difficulty curve through `difficultyAt()`.
