# MAD-Elevator

An endless runner that calculates high scores — vertically, drawn as
pixel art in the Hyper Light Drifter idiom.

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
| hold left / right half | steers that way, like holding an arrow key |
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
| 1,600 ft | 2 · Construction | moving crane booms, swinging jibs with a wrecking ball, falling girders and pallets | 0:29 |
| 3,600 ft | 3 · Skyline | a reprieve: sparse birds, landmark towers in the distance, and both power-ups | 0:51 |
| 5,600 ft | 4 · Low Airspace | helicopters and light aircraft, then jets from 7,200 ft | 1:06 |
| 8,800 ft | 5 · Upper Atmosphere | a meteor shower; from 11,200 ft the lights go out and space debris joins in | 1:27 |
| 13,600 ft | 6 · Orbit | debris and satellites, and the moon slides past | 1:55 |
| 18,400 ft | 7 · Asteroid Belt | asteroids, and a starfield that twinkles | 2:18 |
| 23,200 ft | 8 · Interplanetary | asteroids and alien saucers. Mars goes by | 2:39 |
| 28,800 ft | 9 · Deep Space | asteroids, saucers, the occasional comet | 2:57 |

("Reached at" assumes a clean run.) Climb speed steps up zone by zone from 150 to 900 px/s — about 43 to 184
mph — and the spawn interval tightens with it, from roughly one hazard every
2.4s at street level to one every 0.45s out past Mars. A pixel of travel is
worth 0.30 ft, which is what keeps the altimeter readable rather than a
blur; the run itself takes exactly as long as it always did. Squeaking past something scores a near miss and a
small altitude bonus. Your best climb is kept in `localStorage`.

Touch is **directional, not positional**: the car goes the way of whichever
half of the screen you are holding and keeps going while you hold it, rather
than flying to your finger. Steering also tips the car into the turn, by up
to about five degrees in six quantised steps — holding a step keeps the
rotation constant, so the sprite leans without shimmering at this resolution.

## Power-ups

Both pulse red — the only colour in the game, so red always means "this
helps you" — and both carry a hard outline in the normal asset tone, so
they never rely on colour alone to be seen. While one is running a label
sits at the top of the screen; the labels stack, and you can hold both.

**VMS3** — a bubble, handed to you at 4,100 ft in the Skyline zone and
appearing at random every 4,400–7,600 ft after that. It absorbs one
crash: the hit pops the bubble instead of ending the run, and you get 1.3
seconds of mercy to fly clear. Labelled `VMS3 ACTIVATED`.

**F1 System Speed** — a rocket, first at 4,900 ft, then every
4,800–8,400 ft. It lifts the car 267px up the screen and runs
the world 2.6× faster for three seconds, untouchable, then eases it back
down to station — the wind-down is the part that tells you it is ending.
Labelled `F1 SYSTEM SPEED`.

## The look

The game renders into a canvas that is **270×480 device pixels**. Everything
is authored in 540×960 world units and the context is scaled by `1/PIX`, so
the rasteriser itself does the pixelating and CSS blows the result back up
with nearest-neighbour sampling.

The world is built as a **Hyper Light Drifter scene**, which is a matter of
structure before it is a matter of colour.

**Four layers of built silhouette.** You do not fly past a skyline; you fly
up a canyon. Four layers of props run from a hazy monument layer at the back
(`0.10` parallax) through two layers of architecture to a near-black frame at
the very edges (`0.96`) drawn *in front of* everything, the way a Drifter
scene is vignetted by foreground rock. Each prop is a stack of chunky masses
with stepped shoulders, blocks jutting over the lane, antennae with lit tips,
and **big glowing inset panels** — the panels are the detail that carries the
style, and they recede with their layer so the background never reads as
something you could hit. `topUpProps()` keeps each layer stocked as you climb,
so the canyon is continuous.

**What gets built changes with altitude.** City blocks to 10,500 ft, then
towering cloud banks through the airspace zones, then pitted rock and bolted-on
derelicts out in the belt — with angular **crystal growths** on the near layers
throughout, glowing in the zone's accent.

**Materials, not tints.** Every sprite owns a five-stop ramp — outline,
shadow, base, light, specular — authored as a sprite palette would be, with
the zone hue mixed in at only 14%. A crane is orange, a gull is white, the
car is red lacquer with gold trim and blue glass, and the scene still reads
as one place. Light comes from the upper left: the outline pass lays a dark
edge all round, then a light edge up and to the left.

**Built masses, not rectangles.** Every background mass is drawn as an
object — a lit left face, a shaded right face, a cornice with a specular
lip, floor seams every nine pixels and a grid of windows, lit or dark by a
stable hash. That surface detail is the difference between pixel art and a
flat shape, and it is what the eye reads as sixteen-bit.

**One hue family per zone, lit by its complement.** Each zone owns a dominant
hue and an accent roughly opposite it — crimson dusk with cyan lights, plum
construction with amber, a jade skyline with red, magenta upper atmosphere with
cyan. The sky, all four layer tones, the panel lights, the motes and all
twenty-one sprite ramps are generated from that pair in `updatePalette()`; no
sprite owns a colour, only a small bias toward one. Change `ZONE_HUE` and the
whole game changes mood.

**The car.** A gilded lantern, not a box: ball finial over a rounded dome,
a flared two-step cap with corner brackets, faceted blue glazing behind gold
posts and mullions, a decorated skirt carrying a lozenge, and twin thruster
cylinders throwing blue-white plasma. Its shell stays red in every zone; the
trim reads gold, the glass reads sky, and the passenger keeps dark hair, a
pale face and a navy coat whatever the zone hue is doing around them.

**Backlighting and bloom.** The horizon is the brightest thing in frame, so
every sprite takes a dark separation outline plus a bright rim on its
*underside*. Anything that emits gets a radial glow — thrusters, power-ups,
meteors, comets, panel lights, crystals, the horizon itself — and the car
carries a pool of cold plasma light with it up the shaft. It is the lamp of the
scene, the way the campfire is in Drifter's crimson town.

Sprites are flat and hard-edged with three value steps, anything that spins
snaps to quarter turns, and skies are twenty-two flat bands with a dithered
seam between each pair.

Contrast is a fairness requirement, not a finish. Hazards carry the rim light
and a dark outline so they never dissolve into architecture or the black of
space; the car is red in every zone; power-ups pulse red with a bloom; and the
HUD flips its ink and its backing together with the sky.

## Crashing

The run does not simply stop. The car tumbles through a short screen jiggle
and the frame **freezes**, debris hanging in mid-air. A red curtain then
wipes down over the held frame carrying **GAME OVER** on its leading edge,
holds for a beat, and fades to black — and the results come up behind it:
the logo, a quip, the feet travelled, your best, top speed and the zone you
reached, and buttons for riding again or going back to the menu.

## Layout of `index.html`

The script is sectioned in the order it runs: constants, helpers, canvas,
zones, palette, audio, best, state, input, screens, scenery, hazards,
particles, simulation, rendering, main loop, boot.

Tuning knobs worth knowing: `FEET_PER_PX` (how much altitude a pixel of
travel is worth), `SPEED_MIN` / `SPEED_MAX`, `ZONE_TAIL` (how far the last
zone keeps ramping), `SHIELD_AT` / `ROCKET_AT` / `BOOST_*`, `PIX` (the size
of a pixel), `ZONE_HUE` (the hue and accent of every zone), `LAYER_PAR`
and the prop builders, the intro and crash beats (`I_*` and `X_*`), and the `ZONES` table, which maps an altitude to the hazards that spawn there and drives
the difficulty curve through `difficultyAt()`.
