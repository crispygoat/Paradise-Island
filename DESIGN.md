# Paradise Island — Game Design & Simulation Architecture

## Vision

A cozy, relaxing, AFK-friendly game about nurturing your own undiscovered island in
the middle of the Pacific. Tilt-shift diorama presentation, realistic waves and
atmosphere, gorgeous day/night cycles with stars and moon, calm original music.
Ships on Steam. The player never needs to be glued to the screen — the island lives
and grows while they're away.

## The seed

The player enters their **name** and **birth year**. These are hashed (FNV-1a 64-bit
over the lowercased name + year) into a deterministic seed. The seed drives:

1. **Island shape** — heightmap, coastline, coves, a freshwater source, rock outcrops.
2. **Soil map** — where sand, thin soil, and rich volcanic soil sit.
3. **Seed pool** — which plant species (and how many starter seeds of each) this
   island will ever naturally offer. Limited and unique per island.

Same name + year always produces the same island — shareable and personal.

## Core loop

1. **Plant** limited seeds from your pool onto chosen tiles.
2. **Water** plants to boost growth (rain does this for free).
3. **Weather** rolls through: rain, wind, storms. Clouds and rainfall drive growth;
   storms and runoff drive **erosion** on bare terrain (vegetation roots resist it).
4. **Tides** wash things ashore twice a day: new seeds, driftwood, creatures — and
   trash. Trash left to pile up degrades soil and deters wildlife.
5. **Balance**: every species competes. If one plant dominates, it chokes diversity;
   diversity is what attracts wildlife. Herbivores eat vegetation; predators eat
   fauna. Too much of anything tips the island out of balance.
6. **Milestones**: bare rock → pioneer grasses → shrubs/palms → insects → birds →
   small fauna → a **castaway** arrives on a thriving island. Help them survive
   (food, water, shelter come from the ecosystem you built) until rescue.

## AFK design (core pillar)

- The simulation is **deterministic and tick-based** (1 sim tick = 1 in-game minute;
  ecosystem updates every N ticks). On launch, elapsed real time is converted to
  ticks and fast-forwarded — offline progress is exact, not approximated.
- While running, the game idles politely: low tick work, frame cap options, no
  heavy per-frame logic. Cozy players leave games running; don't cook their GPU.
- Neglect never punishes harshly. Unwatered plants grow slowly, not die instantly.
  Trash accumulates gently. Drift, not failure.

## Simulation architecture

A pure data layer, fully separate from rendering (Blueprint now, C++ module later):

- **Grid**: the island is a coarse cell grid (target 128×128). Per cell: elevation,
  soil quality, moisture, temperature, occupant (plant species + growth stage),
  trash level.
- **Plants**: data-table-driven species with growth rate, water need, temperature
  band, spread chance, competition strength, erosion resistance, wildlife appeal.
- **Weather system**: seed-driven weather fronts; clouds carry moisture, drop rain
  by cell; wind direction affects spread and erosion; temperature follows time of
  day + season + weather.
- **Tide system**: twice-daily tide events pick from a loot table (seeds/trash/
  creatures/driftwood) weighted by island state and randomness.
- **Wildlife**: species unlock when diversity/biomass/trash thresholds are met;
  each has food needs (plants or other fauna) that feed back into the grid.
- **Renderer reads, never writes**: PCG scatters vegetation meshes from grid state;
  Water plugin level follows tide phase; sky/fog/post-process follow time +
  temperature + weather.

## Presentation

- **Tilt-shift**: post-process depth of field with narrow focal plane, slight
  saturation boost — the island reads as a living diorama.
- **Atmosphere**: Sky Atmosphere + volumetric clouds + SunPosition-driven sun/moon;
  real star field at night.
- **Temperature cues**: dawn ground fog when cool, heat shimmer/haze in extreme
  heat, exponential height fog tinted by time of day.
- **Ocean**: UE Water plugin; tide level animates through the day; storm states
  raise wave amplitude.
- **Audio**: calm original music (provided by FiveFire Media), layered ambient
  beds (surf, wind, birds) that follow ecosystem richness — the island literally
  sounds more alive as it grows.

## Milestone roadmap

1. **M1 — Island from seed**: name+year → heightmap island mesh (Geometry Script),
   beach/soil zones, surrounding ocean, basic camera + tilt-shift post.
2. **M2 — Living sky**: day/night cycle, sun/moon/stars, weather states, fog and
   temperature cues, tides on the Water body.
3. **M3 — Green**: grid sim + plant species data, planting/watering interaction,
   PCG vegetation scatter from sim state, erosion.
4. **M4 — AFK**: save/load, offline fast-forward, idle performance pass.
5. **M5 — Alive**: tide deliveries (seeds/trash/creatures), trash cleanup,
   wildlife unlock chain, balance pressure.
6. **M6 — The castaway**: arrival event, needs system, rescue arc.
7. **M7 — Ship it**: Steamworks, achievements, cloud saves, settings, music
   integration, polish, store page.
