# Game Build Brief

## 1. Your Role

This is an empty repo. You will design and build a 3D game. You are the **coordinator**:

- Use agents to write code, analyse, test, and review.
- Start a new agent before any agent's context goes over 400K tokens.
- You may run parallel work in separate temporary git worktrees. All of it must be merged into the main worktree in the end.
- Use the `coding` skill.
- Ask me if anything is unclear.

## 2. Scope: Vertical Slice

Build a **vertical slice**: every system in this brief works end to end, with a small amount of content. Content is data (see §5), so adding more later must not need code changes, except for new core behaviours, new mutations, and new species families.

The slice needs enough content to exercise every system, for example:
- 2 biomes and 2 terrain types (so the scanner's exclusive options mean something)
- **3 species families** (see §18), each generating clearly different species, including elites and bosses
- Several mutations, each with its own effect on each family
- A few buildings in each category
- Several cores per slot type and rarity, including at least 2 epic cores
- A small research tree covering the lander, the mothership, and biomass gates
- A full campaign (default: 30 normal + 5 boss waves, tunable)

## 3. Project Documents

Create and maintain these documents under `./docs`. Every agent must read them before working and update them when they change something. Keep each one short. Use simple words.

| File | Contents |
|---|---|
| `architecture.md` | Code architecture: crates, ownership, abstractions, plugins, systems. Current state only, no history. |
| `decisions.md` | Decision register. Only big, high-impact decisions. For each: the question, options, analysis, trade-offs, nuances, and the final decision. |
| `components.md` | Every component: how to use it, expectations, constraints. |
| `glossary.md` | Every domain term used in the docs and the code. One word always means one thing. Never use two words for the same thing. |
| `prompt.md` | This brief, plus any extra information I give you later. |

## 4. Tech Stack and Platforms

- **Engine:** the latest Bevy version on crates.io.
- **Plugins:** any community plugin listed on the Bevy Assets page.
- **Desktop (Windows, Linux, macOS):** the real game, released on Steam.
  - Use the Steam test App ID 480 (Spacewar) for now.
  - Steam cloud saves.
  - Steam achievements. Add only one for now: **stabilize an outpost** (see §14.1). Build the achievement system so adding more is trivial: achievements and their trigger conditions are defined in data, and the Steam backend sits behind an abstraction.
- **Web:** a free version. No Steam. Saves go to local browser storage.

## 5. Data-Driven Design

- Do not hard-code game content. Enemies, buildings, cores, modules, research, achievements, assistant dialogue, attribute values, and procedural generation parameters live in RON asset files that I can read and edit.
- I must be able to tune the game without recompiling. Restarting the game to pick up changes is fine; hot reload is not required.
- Each core type has unique behaviour, so it has its own Rust code. Its parameters must still live in data files. No scripting language.

## 6. Presentation

- 3D game. You choose the visual style.
- **All art and audio are procedural**: meshes, textures, shaders, particles, sound effects, and music are generated in code. No external art or audio files. This includes the company logo on the splash screen.

## 7. Input and Steam Deck

- Support **keyboard and mouse** and **gamepad** fully. Every action must be possible with either.
- The game must work well on the **Steam Deck**: 1280×800, readable UI at that size, and full gamepad play.
- Building placement is on a grid, so a gamepad can move a grid cursor cell by cell.
- During a run the camera is an **RTS camera**: pan, zoom, and rotate.

## 8. Menus

### 8.1 Splash Screen
Shows a company logo, with the Bevy and Rust logos at the bottom. Then goes to the main menu.

### 8.2 Main Menu
- Background: an interesting 3D scene from the game.
- Options: **Play**, **Settings**, **Controls**, **Credits**, **Exit**.

### 8.3 Credits
A scrolling credits screen. The content comes from a RON file so I can edit it.

### 8.4 Play
Opens a popup showing 3 save slots. The player can pick a slot to continue, or reset a slot.

### 8.5 Settings and Controls
- **Settings**: you choose the options (e.g. audio volumes, graphics quality, resolution, window mode, UI scale). Settings are stored separately from save slots.
- **Controls**: shows the bindings for keyboard/mouse and gamepad. Allow rebinding.
- Both are reachable from the main menu and from an in-game **pause menu**, on the mothership and during a run.

### 8.6 Saves
A save holds **mothership progress**: resources, research, unlocks, codex data, kill counts, scanner and causal manipulator state, and all stable outposts.

Each **stable outpost** is saved as its state at the start of its current wave: map seed, the base's full configuration (every building, its position, its slotted cores, power groups, overload settings, build queue, resource beamer settings), its outpost pool, its core inventory, its current wave, and the highest wave reached.

A run that is not yet stable is never saved. Quitting it is the same as losing it.

## 9. Setting and Core Loop

The game is set on a **mothership** that deploys **landers** onto a hostile planet. Landers start bases and extract resources. The goal is to establish a permanent **outpost** on the planet.

1. The mothership deploys a lander.
2. Local fauna (see §18) is drawn to the lander and attacks it in **waves** that get harder over time.
3. The player defends the lander by building a base around it.
4. The lander mines resources into the **outpost pool**. The player spends from it to build during the run. **Resource beamers** send resources up to the **mothership pool**, which pays for research and upgrades (see §10.1).
5. The player controls time: pause, or fast-forward to the next attack.
6. If there are not enough resources for a building, it is **queued**. The player can reorder the build queue. Queued buildings are built as resources become available.
7. Mining takes time, so the player has effectively unlimited time to *design* a base but limited resources to *build* it. Experienced players will likely queue the whole base at the start and fast-forward to the attack.
8. The player is not expected to survive every attack, especially early on. When the lander is destroyed, the run ends. Only resources already beamed up are kept; the outpost pool is lost. Beamed resources pay for **permanent upgrades**, so each new run can build a better, bigger, longer-lasting base.

## 10. Resources

Not all resources are unlocked at the start.

| Resource | Source | Uses |
|---|---|---|
| **Ore** | Mined by the lander. | Basic currency. On the surface: ammo, building, repairing, beaming down blueprints, buying epic cores. On the mothership: research and the causal manipulator. |
| **Biomass** | Killed fauna. Each family's specialty sets which biomass types it drops (see §18.1). | Unlocks gates in the research tree (see §12.1). |
| **Cores** | Dropped by elite enemies, based on their mutations (see §17.3). | Items slotted into buildings (see §17). They only work on the planet surface. |
| **Power** | Power buildings. | Required by all buildings (see §10.2). |
| **Scan resolution** | Surveillance hubs on the surface. | Spent on the scanner in the mothership (see §12.2). |

### 10.1 Outpost Pool and Mothership Pool
- Each outpost has its own **outpost pool** of ore and biomass. The mothership has the **mothership pool**.
- Everything mined or harvested on the surface goes into the outpost pool. Surface costs are paid from the outpost pool only.
- Mothership costs (research, causal manipulator) are paid from the mothership pool only.
- Resources only move **up**. The mothership can never send ore or biomass down. This stops a rich outpost from bootstrapping a new one.
- **Resource beamers** move resources up (see §16). Each resource beamer:
  - beams **one** resource type, chosen by the player
  - has a **retain** amount set by the player; only the amount above it is beamed up
  - has a maximum **throughput** (amount per second)
  - uses power, like any building
- In a run that is not stable, beamed resources reach the mothership pool immediately. When the run is lost (lander destroyed, player gives up, or player quits), beamed resources are kept and the outpost pool is lost.
- In a stable outpost, resources beamed during a wave only reach the mothership pool when the wave is cleared (see §14.3).
- Scan resolution goes straight to the mothership pool; it has no use on the surface.

### 10.2 Power
- A building's effectiveness depends on how much power it gets.
- Some buildings generate power; some store it.
- The player can group buildings and directly control how much power each group gets.
- Buildings can be **overloaded**: better performance, but more power use and damage over time.
- A damaged building is less efficient.

## 11. Mothership Scene

- This is the first scene the player sees after choosing a save slot.
- 3D scene, fixed first-person view of the mothership control panel.
- A **holographic assistant** appears on the left. It gives the story and tips through a fixed dialogue tree, defined in data. The topics available depend on what the player has unlocked.
- A button launches the lander.
- More buttons unlock over time:
  - **Research**: unlocked after the first run ends.
  - **Codex**, **Loadout**, **Scanner**, **Causal manipulator**: unlocked through research.
- The **codex** shows everything discovered: cores, enemies, buildings, etc.

## 12. Mothership Features

### 12.1 Research
One research tree with two branches:
1. **Lander upgrades** (see §13)
2. **Mothership upgrades**: unlocking mothership features and upgrading the scanner and causal manipulator

Ore pays for each research node. Some nodes sit behind **gates** that must first be unlocked with a specific type of biomass.

### 12.2 Scanner
The scanner influences the game's randomness. It "scans" for good landing sites, e.g. more of a specific biomass type, more of a specific core, or a specific biome or terrain.

- It costs **scan resolution**.
- **Exclusive** options (e.g. biome, terrain): only one per group can be picked; the result is guaranteed.
- **Stackable** options (e.g. higher drop rate of a core, more of a biomass type): increase chances and can be combined. Picking a biomass type makes waves favour families whose drop tables include it. Picking a core makes waves favour mutations related to it.
- Deselecting an option refunds its scan resolution.

### 12.3 Causal Manipulator
Permanently removes things from the game, such as a core, a mutation, or an enemy trait.

- Costs ore. Each use is one-time.
- It is deliberately very expensive. The purpose is to spend a lot to break out of a plateau by giving future runs a bigger edge.

## 13. Loadout and Lander

- The lander always arrives with **zero ore**. It mines ore and produces a base amount of power on its own, and buildings from its building modules are already deployed (see below). All of these values are in data and can be raised by research.
- Before launch, the player fits the lander with a **loadout**. The first launch always uses the starter loadout.
- The loadout is made of **modules** slotted into the lander. There are two kinds:
  - **Building modules**: the building is deployed instantly on landing. Uses the lander's **power rating**; the total cannot exceed it.
  - **Blueprint modules**: the building can be built after mining enough resources. Uses the lander's **CPU rating**; the total cannot exceed it.
- Building slots and blueprint slots are separate, and each has a maximum count.
- Research can upgrade CPU, power, and slot counts.
- Modules are unlocked through research. Advanced modules also need biomass gates.
- During a run, blueprints not in the loadout can be **beamed down** from the mothership. They cost ore from the outpost pool.

## 14. The Main Game (A Run)

- On launch, a **procedurally generated arena** is created. The lander lands at the centre.
- The map has spawn points that change between waves.
- The player gives build orders, then waits or fast-forwards to the start of the wave.
- During a wave the player can still mine, gain resources, build, and beam down blueprints.
- Research, loadout, scanner, and causal manipulator are only available on the mothership, never during a run.
- The player can give up at any time. Quitting a run that is not yet stable, even mid-wave, is the same as losing it.
- Each kill drops biomass from its family's drop table into the outpost pool. Kills are tracked in the codex (see §18.4).
- When a core drops, the player picks one of 3 random options. It goes into the run's inventory and can be slotted into a matching building. Unslotted cores do nothing. Cores stay on the planet: when the run ends, its cores are gone.

### 14.1 Waves
- Every wave is produced by the **wave planner** (see §14.2). No enemy has hand-written stats.
- A **boss wave** happens every X waves (tunable).
- A campaign has a finite number of waves (default: 30 normal + 5 boss waves). Clearing them makes the outpost **stable** and unlocks the achievement.

### 14.2 Wave Planner
The wave planner makes difficulty adapt to what the player could have built, then pushes the player towards using cores.

1. **Forecast.** At the start of each wave, it estimates the **forecast DPS**: the DPS the player could reach *without cores*, using only:
   - the research tree state (unlocked modules, buildings, and upgrades), and
   - the total ore mined in this run so far. Use ore *mined*, not the outpost pool, so beaming ore up does not make waves easier.

   Power is not part of the forecast.

   Do **not** use hand-written formulas; they drift as content changes. Instead, generate many different base configurations that the research state and ore allow, measure the theoretical DPS of each, and use the resulting spread of values (e.g. low, median, and high percentiles) as the forecast.
2. **Plan.** It produces a **wave plan** from:
   - the forecast
   - the wave number
   - **context**: the styles of previous waves, so that over a run it covers all the different styles of attack instead of repeating one
   - scanner choices (see §12.2)
3. **Requirements.** A wave plan is a set of requirements the wave must meet, for example:
   - **required DPS** to clear it
   - **required damage projection**: how far the player's weapons must reach
   - other requirements you find useful (e.g. handling fast enemies, burrowers, or swarms)
4. **Generate.** The species generator and the spawner turn the wave plan into species, mutations, numbers, spawn points, and timing that meet the requirements (see §18).

**Difficulty curve.** Where the required DPS sits relative to the forecast follows a curve set in RON:
- Early waves stay inside the forecast, near the low end.
- As waves go up, it moves towards the high end.
- Later it goes beyond the forecast, then to several times it. Cores can multiply output, so by then the player is expected to choose and combine cores well to keep up.
- Difficulty beyond the forecast comes mainly from **mutations** on the enemies (see §18.3), not just more or bigger enemies.
- Stable outposts continue the curve, but it grows very slowly.

The forecast and the wave planner are the same code that the balancing tools use (see §19).

### 14.3 Stable Outposts
- Stable outposts are the player's long-term **frontiers**: builds they want to keep working on and pushing, like having several characters in an RPG. The player can have many and return to any of them at any time.
- A stable outpost has infinite waves that grow stronger very slowly, so the player can push it as far as they can.
- A stable outpost **can never be lost**. Losing a wave, or quitting during one, undoes the wave completely, as if it never happened: the base, the outpost pool, cores, kills, and resources beamed during that wave are all restored to their state at the start of the wave.
- Only a cleared wave changes anything. Stable outposts do nothing while the player is away.
- For each stable outpost, the game records its current wave and the highest wave reached.

## 15. Map Generation

This is a **tower defense roguelite**. Map generation is critical to the game: it is what makes each run play differently and makes base design interesting. Give it real design effort.

- Maps are built on the same grid as building placement.
- Terrain must create interesting tactical layouts: **choke points**, **multiple paths** to the lander, open killing fields, high ground, and areas that cannot be built on.
- Spawn points change between waves, so a base that only covers one approach should be at risk.
- Enemies move through terrain in different ways (see §18). Walling in the base is never enough on its own.
- Biome and terrain type (chosen by the scanner, see §12.2) change the generated layout, not just the look.
- Every generated map must be valid: each spawn point can reach the lander, and the lander has enough buildable space around it. Validate this in code and reject or repair bad maps.
- All generation parameters live in RON files.
- The same seed always produces the same map. Stable outposts depend on this.

## 16. Buildings

| Category | Purpose | Examples |
|---|---|---|
| **Offensive** | Kill enemies. | Guns, lasers, plasma, artillery. |
| **Resource** | Produce a lasting resource, or boost gathering. Investments: trade early power for big later gains, in this run or the next. | Surveillance hub (scan resolution), biomass processor, ore waste processor. |
| **Defensive** | Protect or steer enemies, as meat shields or to funnel enemies into offensive buildings. | Walls; deflectors that use power to push up to a set mass at intervals. |
| **Power** | Produce or store power. | Generators; batteries for efficiency and short power bursts. |
| **Logistics** | Move resources between the outpost and the mothership. | Resource beamer (see §10.1). |

## 17. Cores

### 17.1 Slots
Buildings have up to 3 slot types:

| Slot | Effect | Examples |
|---|---|---|
| **High** | Boosts the building's attack. | Damage, fire rate, penetration, homing. |
| **Mid** | Acts on enemies directly. | Slow in a radius, higher biomass drops, lower enemy resistances in a radius. |
| **Low** | Boosts the building itself. | Yield, lower power use, more HP. |

Typical layouts:
- Offensive: high + low
- Defensive: mid + low
- Resource and power: mostly low
- Special buildings: all 3

Cores can also have negative effects.

### 17.2 Rarities
- **Normal**: most common.
- **Rare**: drops more often from stronger enemies.
- **Epic**: rarest. Reliably earned only by completing an optional objective during a wave (e.g. kill a specific optional enemy, lose no more than X buildings, don't use a certain weapon). On completion the player gets the usual 3 choices, but must pay ore for the chosen core; the price depends on the core.

Epic cores change the game and should shape the player's strategy. Examples:
- **Cluster bomb**: the projectile keeps firing copies of the building's projectile, inspired by bullet-hell games.
- **Extreme piercing**: shots pierce enemies, terrain, *and your own base*, so the building must be aimed carefully.
- **Terrain changers**: double-edged effects that reshape the map.

### 17.3 Drop Rules
- Cores come from enemy **mutations** (see §18.3). Each mutation lists its related cores in RON. When an elite drops a core, the choices are drawn mostly from cores related to that species' mutations, so the player can see what a species carries and hunt for it.
- Every drop is still a random selection.
- Some cores can be slotted into the lander to increase the number of choices offered, or how many the player may pick.
- A "save core" ability lets the player keep unpicked cores; they appear again in the next drop, so good cores from one drop are not lost.
- A core is never destroyed. If its building is destroyed, it returns to the inventory.

## 18. Enemies

This is **not** a classic tower defense with passive enemies that walk a path. Every enemy fights, and the game should feel like an arms race between buildings and enemies. Every enemy is aggressive towards the player; aggression is not a trait.

Enemies differ in:
- **Attack type and effective range**: melee, ranged, artillery, area, etc.
- **Target priority**: e.g. the lander, power buildings, offensive buildings, the nearest building.
- **Movement and pathing**: e.g. ground, burrowing, climbing, flying, wall-breaking.
- **Speed**
- **Group behaviour**: herding, swarming, packs, loners.
- **Interactions with other families**: e.g. one buffs, follows, or feeds on another.

Walling in the base must never be enough on its own.

### 18.1 Species Families
- There is a **fixed set** of species families, defined in data and code. The vertical slice has **3**.
- A family defines the enemy's basics: how it moves, its base model, and its animations.
- A family defines **tendencies**, e.g. prefers long-range attacks, tends to be fast.
- A family defines the **limits** of each attribute, which species generated from it must stay within.
- A family has a **specialty**, which sets its **drop table**. Biomass types are a fixed list in data, so research gates and the scanner can name them.

### 18.2 Species
- A **species** is one specific, procedurally generated instance of a family. New species are generated for each wave to meet its wave plan (see §14.2).
- A species is defined by its **genome**: the generated data that fully describes it, including its family, attributes, and mutations. The genome's **hash** is the species' identity.
- The hash must be stable across platforms and game versions: hash a canonical serialization of the genome with a fixed algorithm (not Rust's default hasher).
- A genome includes final values. Tuning the RON changes what the generator produces next, not genomes that already exist.
- **Every species looks unique.** Its attributes and mutations feed into its hash, and the hash drives its appearance: procedural variation on the family's base model, such as colour, pattern, proportions, and extra body parts. The same hash always looks the same.
- Elites and bosses are species with more and stronger mutations.
- Sharing species between players is out of scope for now. Keep it possible later.

### 18.3 Mutations
- **Mutations** are the enemy version of cores: they enhance a species in different ways. Mutations are procedurally chosen and rolled for each species.
- Mutations are equipped on the **species**, so every spawned member of that species has them.
- The game is worse than the player at picking good combinations, so enemies get **more mutations** than buildings get cores.
- Mutations are how the player gets cores: elites drop cores related to their mutations (see §17.3).
- Each mutation affects each family differently, just as a core affects each building differently. So each mutation has its own code per family. Its parameters live in RON.

### 18.4 Codex
- The codex tracks kill counts per **family**, per run and all-time.
- It lists every **species** discovered, with its genome and hash. Having both is the proof that the player discovered it.

## 19. Testing and Tooling

- Unit tests for most of the logic.
- **Balancing tools** that run without the game window, reading the same RON data as the game. At least:
  - **Wave demand**: the wave plan and required DPS for a given wave.
  - **Build output**: the forecast for a given research state and amount of ore, including the best and worst sampled base configurations.
  - **Difficulty curve**: print the forecast DPS and required DPS for waves 1 to N, for a given research state and mining rate.
  - **Core comparison**: the effect of a core, or a set of cores, on a building's output.
  - **Mutation comparison**: the effect of a mutation, or a set of mutations, on each family.
- **Species generation tools**: generate species from a seed and a wave plan, and print their genome, hash, attributes, and mutations, so I can check the variety and difficulty.
- **Map generation tools**: generate maps from a seed and parameters without the game window. Output a top-down preview image and metrics such as the number of paths, the number of choke points, path lengths from each spawn point, and buildable area, so I can tune the generator.
- **Debug tools** in the game, for example: grant resources, skip to a wave, spawn enemies, and show stats such as DPS, power, and the build queue.
