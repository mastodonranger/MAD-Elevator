# MAD-Elevator

A vertical endless runner in a 9:16 frame. Car 3 has left the building and
it is not coming back down. You steer left and right; the only number that
matters is how many feet you made before something hit you.

Everything is one self-contained file — `index.html`. No build, no assets,
no network requests: the art is drawn with the canvas 2D API at runtime.

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
| `Esc` / `P` | pause (on the results screen, back to menu) |
| `R` | restart mid-run |
| `M` | mute |

## Altitude is on a real scale

The car is 118px tall and reads as an 8ft elevator, so a pixel of climb is
`0.07ft`. That is what the whole game is paced against: the world's tallest
towers top out around 2,500ft, and it takes about two minutes of flying to
get above them.

**Every zone is 500ft**, and a zone is a stretch of flying rather than a
couple of seconds — about 28 seconds at launch, tightening to roughly 12 as
the climb speed ramps from 240 to 620 px/s.

| From | Zone | What is up there |
| --- | --- | --- |
| 0 ft | Street Level | pigeons, and the city right outside the glass |
| 500 ft | Midtown | birds, the first crane booms |
| 1,000 ft | High Rise | crane booms with one passable gap |
| 1,500 ft | Skyline | cranes, and what falls off them |
| 2,000 ft | Spire | the top of the tallest towers ever built |
| 2,500 ft | Rooftop Winds | construction thins out, the first helicopter |
| 3,000 ft | Low Airspace | helicopters and birds — nothing built reaches here |
| 3,500 ft | Helicopter Lanes | news, police and rescue traffic |
| 4,000 ft | Approach | helicopters, and airliners on the way in |
| 4,500 ft | Flight Level | airliners |
| 5,000 ft | Cruising Altitude | airliners, the last few helicopters |
| 5,500 ft | Stratosphere | the first meteors coming in to burn |
| 6,000 ft | Mesosphere | meteors |
| 6,500 ft | Meteor Shower | a lot of meteors |
| 7,000 ft | Thermosphere | meteors, and the first wreckage in orbit |
| 7,500 ft | Kármán Line | satellite wreckage and asteroids |
| 8,000 ft | Low Orbit | whole satellites among the debris |
| 8,500 ft | Debris Belt | torn panels, dishes, truss |
| 9,000 ft | Asteroid Field | asteroids |
| 9,500 ft | Deep Space | asteroids, and something else out there |
| 10,000 ft | The Long Dark | saucers, rock, and whatever is left in orbit |

**Nothing built appears above 2,800ft.** That is enforced at the spawner,
not just implied by the zone table, so a crane or a falling girder can never
turn up in the airspace zones whatever the mix says.

Above that it is only things that fly, then only things burning up, then
only what is in orbit. Squeaking past something scores a near miss and a
small altitude bonus. Your best climb is kept in `localStorage`.

Touch is **directional, not positional**: the car goes the way of whichever
half of the screen you are holding and keeps going while you hold it, rather
than flying to your finger.

## How the pacing holds together

Two things have to track the climb speed, or slowing the game down quietly
makes it harder instead of calmer:

**Hazard spacing is a distance, not a delay.** The gap between one hazard
and the next is `470px` of climb at launch, tightening to `200px` — so the
vertical spacing on screen is the same whatever the speed. Timed spawning
would have packed hazards a third closer the moment the climb slowed.

**Sideways motion scales with the climb.** A hazard that drifts across the
lane at a fixed px/s covers much more ground during a slow approach than a
fast one, which turns dodging into luck. Bird drift and sway, saucer homing,
and the lateral speed of everything that wanders are all set against the
current speed range.

Measured with an autopilot: the same bot survives about 30 seconds a run
here against about 9 before the change.

## Crashing

The run does not simply stop. The car tumbles through a short screen jiggle
and the frame **freezes**, debris hanging in mid-air. A red curtain wipes
down over the held frame carrying **GAME OVER**, holds for a beat, and fades
to black — and the results come up behind it.

## Layout of `index.html`

The script is sectioned in the order it runs: constants, helpers, canvas,
zones, palette, audio, best, state, input, screens, scenery, hazards,
particles, simulation, rendering, main loop, boot.

Tuning knobs worth knowing: `FEET_PER_PX` (how much altitude a pixel of
travel is worth), `SPEED_MIN` / `SPEED_MAX`, `RAMP_FEET` (where difficulty
maxes out), `SPAWN_GAP_EASY` / `SPAWN_GAP_HARD`, `ZONE_H`,
`CONSTRUCTION_TOP`, the `ZONES` table, `SKY`, and `cloudDensity()`.
