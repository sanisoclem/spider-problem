this is an empty repo. I want you to design and build a game, you are the coordinator. Use agents to build the code, do analysis, testing, etc. Start new agents whenever context is going to go over 400K. You can start parallel work in different temporary worktrees but they all have to be merged to the main worktree eventually.

I want you to create and maintain the following documents (under ./docs) and make sure it is referenced/updated by agents to keep everyone on the same page. Keep all documents short and sweet as reasonably possible, use simple words:

- architecture.md - describes the code architecture, crates, ownership, abstractions, plugins, systems, etc. Do not include historical commentary here
- decision register - this is only for big impactful decisions, put the question, options, analysis, tradeoffs, nuances and the final decision here.
- components.md - a list of all components and how they should be used, expectations, contraints etc
- glossary.md - all words used in the docs and in code must be documented here. You must prioritise consistency and integrity - one word should always describe the same thing, do not use multiple words to describe the same thing.
- prompt.md - put this initial prompt and any additional data I give you

use the coding skill

The tech stack:

- use the latest version of bevy available in crates.io
- you can use any of the community plugins available in the website
- must target windows, linux, mac and web.
- must integrate with steam, with achievements and uploading cloud saves

Menu:

The game must start with a splash screen showing a company logo including bevy and rust in the bottom then go to the main menu. The main menu should be an interesting 3d scene from the game, with 3 options - Play, Credits and Exit. Credits just shows a scrolling credits screen that can be customized by editing an assets (preferrably ron). Play opens a popup that shows all saves (3 slots). The user can select a save to continue or reset a save.

Setting:

The game is set on a mothership that is deploying landers on a hostile planet. The landers are base starters and extractors. The whole point of the game is to establish a permanent outpost on the planet. The mothership deploys a lander, however, the local fauna is hostile and is attracted to the lander, they attack it in waves that get progressively difficult. The player must defend the lander by building a base around it. The lander mines resources from the planet, and these resources can be sent back to the mothership or used for base building. The player can control passage of time, pause or fast forward to the attack. They can also build a base. If there are no resources available to build something. that build is queued. The player can then manipulate the build queue to reorder things. Buildings are built as resources are available. Since mining resources takes time, the player has effectively unlimited time to design and build their base but has limited resources. I expect players who are accustomed to the initial waves would build all the buildings at the start, queued, and then fast forward the clock to the attack. The player is not expected to survive all the attacks, especially in the beginning, but will be able to send resources to the mothership that can be used for permanent upgrades. These upgrades help the player on their next excursion to the planet, so they can progressively build better, bigger and longer lasting bases.

The Resources:

The currencies of the game. The player will gain these in different ways and are not all unlocked at the same time.

- ore - the basic currency, it used to make ammo, can be sent to the mothership, can be used to build and repair buildings. This resource is mined by the lander. This can be used for research to improve the lander and mothership.
- biomass - harvested from killed fauna. This is used to unlock nodes in the tech tree. each type of fauna produces its own type of biomass
- cores - dropped by elite enemies. These are items that can be slotted into different buildings for various effects. These can simply augment the building or completely change its behaviour. cores come in different rarities and types. offensive cores are slotted into offensive buildings to change how they operate. utility cores can be slotted to any building and provides some passive effect. cores can also have associated negative effects.
- power - basically electricity. buildings require power and their effectives is determined by how much power they have. some buildings generate or store power. The player is able to group buildings to directly control how much power is given to buildings. Buildings can also be overloaded for better performance at the cost of more power and damage over time. Building damage also decreases building efficiency.

The mothership:

It is the scene that is displayed when the player first enters the game, it should show the mothership control panel. The player has a holographic assistant that shows on the left that gives them the narrative and tips and answers any questions. The player then has access to a button to launch the lander. This is all a 3d scene and the player has a fixed first person view). After the player unlocks it they can also unlock the buttons to open research, the pokedex (shows all cores discovered, enemies, buildings, everything), the loadout, the scanner and the causal manipulator.

Research

is composed of different trees:

1. lander upgrades - (see lander section below)
2. mothership upgrades - (see motership section)

Loadout/Lander

The lander is fitted with a loadout before it is launched, except for the first one where it is launched with the starter loadout. The loadout determines which buildings are included in the lander and can be deployed instantly upon landing, which buildings can be build after mining the necessary resources. Blueprints can also be beamed down later even if not in the loadout but this requires resources from the surface. The lander has a power rating and CPU rating. buildings included in the lander use up power, and cannot exceed the power rating. blueprints included in the lander use CPU, and cannot exceed it. The lander can be upgraded with research. Buildings and blueprints are packaged into "modules" that are slotted into the lander, the slots for blueprints and buildings are different, they both have a max number of slots. The CPU, power, number of slots can all be upgraded with research. The modules themselves are also unlocked with research but the more advanced ones are gated by specific biomass

Mothership

The mothership has a few functions. First is the scanner, the scanner influences the RNG of the game, it "scans" for favorable locations so that the player can find a specific species of fauna, or more of a specific core, or more favorable ground conditions, like a specific biome, or terrain. This is done by consuming scan resolution, which is currency that is earned by running surveilance on the surface by building a surveilance hub. This can used in the mothership to unlock the scanner and to select various conditions for the next launch. Some selections are exclusive, like the biome or terrain.. and are guaranteed... some just increases the chances and can be combined, like increased drop rate for a certain core or to see a specific species of enemy. The scan resolution is returned if an option is deselected. The other feature is the causal manipulator, which makes some things never appear, like removing cores or enemies from the game.. these use actual ore and are one time... so excluding something is very expensive and the idea is to use up a lot of resources to potentially break out of a plateau by increasing the advantages of a single run.

The main game:
After launch, procedurally generated arena is created. The lander lands at the center of the map. The map also has various spawn points that change between attack waves. players issue build instructions and wait/fast forward to wave attack start. while the attack is ongoing, the player will still be mining, getting resources, can continue building, beam blueprints from the mothership. They cannot do research or customize the loadoat or scanner or causal manipulator, these only done in the mothership. the player can give up at any time. monster kills produce a biomass currency for that monster. monster kills for that species are also tracked per run and all time and is recorded in the pokedex. when a core drops, the player is presented with 3 options and can pick the core that they want. This core is then placed in their inventory and can be slotted to the applicable buildings. unslotted cores don't do anything.

Buildings

Offensive - these are like guns, lasers, plasma, artillery, etc.

Resource building - produces some permanent resource like scan resolution. Or they can increase efficiency of specific resource gathering, like biomass processing facilities which increase the collection of biomass or ore waste processing to increase ore mining yield.. these are investments and trade early power for exponential future gains (either in the same run or the next)

defensive - produce some kind of defensive effect, like walls, or deflectors that uses power to move up to a certain amount of mass every so often.. these can be used as either meat shields or to manipulate the enemy to make the offensive buildings more effective.

power - produces power, everything needs power. also includes things like battery storage for more effecient use of power and to allow surgical power bursts.

Cores

Cores are items the player equips to certain slots in buildings. There are 3 types of slots, high slots, mid slots and low slots. Most offensive buildings will have high and low. most defensive buildings will have low and mid.. resource and power buildings will mostly have low slots. Some special buildings will have all 3.

high slots augment the offensive capability of the building - like adding damage, increasing fire rate, increasing penetration, making it homing, etc.

mid slots do something to the enemy directly - like slowing the enemy within a radius, or increasing biomass drop rate, or lowering defense/resistances in a radius.

low slots affect the building itself, increasing yield, less power consumption, increased hp etc

cores also have different rarities, normal cores are the most common, rare cores drop more the stronger the monster is, epic cores are the rarest, and can only be reliably obtained by completing a specific objective in a wave (like killing specific optional enemy, or not losing more than X buildings, or not using a specific weapon, etc). Upon completing that objective, the player is given the standard 3 choices of epic cores but they also have to pay for it, with the cost dependent on the core in question. Epic cores are game changing and will usually influence the player's strategy a lot. epic cores include something like a "cluster bomb", where it shoots a projectile that continuously shoots the same projectile as the building.. heavily inspired by bullet hell games.. or maybe makes the attack very incredible piercing, so much so that you have to orient it correctly, because it pierces through the enemy, terrain and even your base. And yes, epic cores could do things like change terrain so it could also be a double edged swor.

When a core is dropped/obtained, it is always a random selection. Some cores can be slotted to the lander to increase the number of "core choices" or the number of cores you can pick from the selection. another one is the ability to "save" a core for later, so when another one drops, the "saved cores" show up in the same slot so if 3 good cores drop in one go, you can still get them on the next ones.

A cores is not destroyed when a building is destroyed, it returns to your inventory.

enemies

I leave the design of this up to you, it really should be an arms race between the buildings and the enemies

waves

The waves are procedurally generated, but have some parameters that can be tuned. waves get harder and harder and have a boss wave every X waves (tunable). The main game only has a finite amount of waves, maybe 30 normal waves + 5 boss waves. and once that is clear, the outpost is "stable". A stable outpost is just for collection purposes (the player can return to it anytime and play it). it will have infinite waves that still get progressively stronger but only very little so the player can keep playing as much as they want (in theory), the player could still technically lose the outpost.

Design
You have to include audio, art, shaders, particles etc. This is a 3d game. I leave the visual style up to you.

Make it data driven, don't hard code monsters, buildings in code. Make them resources, preferrably ron files so I can read and edit them. Make sure that we can tune the game without having to recompile. Things like cores, buildings, attribute values, procedural generation parameters should not live in code.. Since cores have very unique effects, each core type has specific code attached to it. But the parameters must still be tunable. I am open to also scripting these, if that doesn't cost much to implement. but I am fine with also just writing them in rust

Ask if you have any questions.
