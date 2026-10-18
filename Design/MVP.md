# Earthrise — MVP Design

**Status:** Draft 0.2 — 2026-10-18. Built from the owner interviews of the same date; every decision below was taken by the owner unless it is listed under §12 *Open questions*, where it is a proposal awaiting an answer.

**Authority.** This document says *what* is built. [`AGENTS.md`](../AGENTS.md) says *how* code is written, and `Design/ADR/` records the engineering decisions taken while building. Where this document needs an engineering decision it names it in §11 and leaves the decision to an ADR. A task that needs a design answer this document does not give asks the owner, and the answer is written here before the code is.

**Companion:** [`Design/PROMPT.md`](PROMPT.md) is the condensed brief for seeding a new session. When the two disagree, this document wins and the brief is stale.

---

## 1. Vision

Earthrise is a massive-scale, three-faction territory war in the mould of PlanetSide 2, set in a voxel world where the bases being fought over can be built up, blown apart and rebuilt while the fight is on. The concept art's tagline is the pillar statement: **build, fight, conquer.** In the long run that means hundreds of players per continent, ground and air vehicles, dedicated servers, persistent territory and progression.

The MVP is the smallest thing that proves the three pillars hold together and that the hardest technology behind them works at scale:

> One human, in first person, fights in an endless three-faction territory war on a 2×2 km map, alongside and against about a hundred bots, over voxel bases that every faction — bots included — builds, breaches and repairs. The server is a separate process the client launches, and nothing the client does bypasses the network protocol.

What makes PlanetSide 2 what it is — scale — cannot be proven with one human. It *can* be proven on the server, which is where it is hard: a hundred bots exercise simulation cost, interest management, voxel edit replication and crowd rendering exactly as a hundred humans would, and they do it on demand, unattended, for as long as a soak test runs.

## 2. What we take from PlanetSide 2, and what we do not

PlanetSide 2 is a reference for mechanics and scale. Its names, factions, fiction, assets and art are not used.

| PlanetSide 2 | MVP | Later | Note |
|---|---|---|---|
| Three factions | ✔ symmetric | Asymmetric weapon philosophies | Data model is N-faction from the first line (D-05) |
| Lattice territory, warpgates | ✔ | Cut-off rules, continent locks | |
| Bases with control points | ✔ 1–3 points per base | Generators, shield doors, base turrets | |
| Six infantry classes | 3 classes | Infiltrator, jump-jet assault, heavy suit | |
| Ground and air vehicles | ✘ | ✔ | Seat/possession seam kept open (§6.13) |
| Mobile spawns (Sunderer, beacons) | ✘ | ✔ | |
| Construction system (Cortium, modules) | ✔ voxel modules + repair | Free-form building, harvesting | Reference for D-22 and D-25 |
| Continent scale (~8×8 km) | 2×2 km | Larger continents | Coordinates already fit 8 km (§7.2) |
| Hundreds of humans | 1 human + 99 bots | Real multiplayer | Second client connectable in dev (D-02) |
| Certification points, unlocks | ✘ | ✔ | Nothing persists (D-11) |
| Client-side hit detection | **Never** | — | Deliberate departure (D-10) |
| Third-person infantry camera | **Never** | — | First-person only (D-19) |

## 3. Decision log

Each row is the owner's answer and what it costs or forecloses. Numbering is permanent; a reversed decision gets a new row that supersedes the old one, it is not edited in place.

| # | Decision | Consequence |
|---|---|---|
| D-01 | **The server is a separate headless executable** that the client launches as a child process and talks to over loopback UDP. The same binary is the future dedicated server. | Netcode is real from the first build. No shared memory between client and server, ever. Loopback is too kind, so a network conditioner is mandatory (§7.4). |
| D-02 | **Exactly one human, but net-ready.** All traffic goes through the client/server protocol. | A second client must be able to join from a dev command line without a code change — that is the acceptance test for "net-ready" (§9). |
| D-03 | **Dependencies per AGENTS.md R14:** Windows SDK and MSVC standard library only, in anything that ships. | Physics, collision, navigation, netcode, UI and text are hand-written. Offline tools under `Tools/` are exempt and may use third-party code. |
| D-04 | **The MVP proves territory + bot scale + voxel build/destroy.** | About 100 soldiers on the server; every system is sized and measured for that (§9). |
| D-05 | **Three factions.** Player's faction plus two AI factions that also fight each other. | Nothing is two-sided: no `isEnemy` booleans, relationships by faction id, faction count in data (R23). |
| D-06 | **Infantry only.** | No vehicle dynamics in the MVP; the entity and input model keep a possession seam. |
| D-07 | **Map about 2×2 km.** | One map file, no streaming. Distant bases need lower-detail meshes (§7.6). |
| D-08 | **Three infantry classes.** | Kits in §6.5 (D-30). |
| D-09 | **Endless sandbox,** no win state. | Flow must be sustained by rules, not by a round reset (D-16), and "done" is measured by flow metrics, not by a winner (§9). |
| D-10 | **Server-authoritative hits with lag compensation.** | Server keeps hitbox history and rewinds; projectiles simulated on the server. Client predicts its own movement and shows provisional effects only. |
| D-11 | **Nothing persists** between sessions except settings. | No save format, no economy beyond the in-session build pool. |
| D-12 | **Windows 11, modern GPU,** inbox D3D12 runtime only (no Agility SDK, R14). | What the inbox runtime exposes is verified on day one and recorded (ADR backlog §11). |
| D-13 | **Stylised, self-made art.** Voxels are for buildings and, where it helps the look, parts of the environment; soldiers, weapons and later vehicles are conventional low-poly meshes, not voxels. | Two content paths: voxel blueprints for the world, meshes for everything that moves. Soldiers animate as rigid parts (D-40). |
| D-14 | **Symmetric factions.** Faction differs by colour and identity only. | Map and AI balance are isolated from weapon balance. |
| D-15 | **Autonomous allies.** The player issues no orders. | Faction AI decides objectives; a map ping is cosmetic in the MVP. |
| D-16 | **Warpgates plus a population director.** | Each faction has an uncapturable warpgate; the director nudges the losing faction within fixed bounds (§6.10). |
| D-17 | **Spawn at the warpgate or an owned base;** no mobile spawns. | A base under attack keeps its spawn until the attackers hold all its points (D-28). |
| D-18 | **Lattice with control points,** 1–3 per base. | Capture rules in §6.4. |
| D-19 | **First-person only.** | Own body is arms and weapon; others are full-body models. |
| D-20 | **Voxel bases on a column heightfield.** Bases and fortifications are voxel volumes; terrain is a blocky heightfield on the same grid and is not destructible. | No caves or overhangs in terrain. One collision model for both (§7.3). |
| D-21 | **Voxel size 0.5 m.** | A soldier is ~3.6 voxels tall; doorway, stair and step rules follow from it (§6.6). |
| D-22 | **Construction is blueprint modules plus repair.** Engineers place prefab modules on build slots in owned bases and repair damage back to blueprint. | No free-form block placement. Bots can make every construction choice a human can. |
| D-23 | **Fortifications are destroyed by explosives only.** Spawn rooms, control points, terminals and warpgates are hardened. | Small arms never edit voxels. The capture loop cannot be deleted. |
| D-24 | **Bases restore slowly when calm;** engineers repair much faster. | The map heals between fights; damage matters during them. |
| D-25 | **Construction is paid from a territory-fed faction pool.** | Territory and building feed each other; the director gets an income lever. |
| D-26 | **Unsupported voxels crumble.** | Bounded connectivity check after every edit; no rigid bodies (§7.3). |
| D-27 | **Bots build, repair and breach.** | Bots use the voxel mechanic as fully as the human does; this is the MVP's largest AI risk (§10). |
| D-28 | **Defenders keep a base's spawn until the attackers hold every one of its control points** (O-01). | A full sweep of the points is the decisive moment of an attack, and a defence does not have to walk in from the next base. |
| D-29 | **Three warpgates and seven bases,** rotationally symmetric, as in §5.1 (O-02). | Seven blueprints to author: one large, three medium, three small. |
| D-30 | **Class kits as in §6.5:** assault breaches, medic heals and revives, engineer builds, repairs and resupplies (O-03). | Each voxel job belongs to one class. No jump jets, so navigation needs no jump links beyond two-voxel ledges. |
| D-31 | **Friendly fire from explosives only** (O-04). Grenades and breach charges hurt allies; bullets pass through them. | One human among 32 friendly bots is not teamkilled by bot aim error. It is a data flag. Bots must check a blast against nearby allies before throwing or placing. |
| D-32 | **Modules crumble when a base changes owner** (O-05). The new owner starts from the bare shell. | Fortifying a base is a bet that it holds, so the commander weighs a module's cost against the risk of losing the base. A capture is the biggest single voxel event in the game; its remeshing and navigation rebuild are spread over several ticks. |
| D-33 | **The tick rate starts at 30 Hz** and is settled by measurement (O-06). | Measured at M3 and M6, recorded in ADR-004. Commands carry exact fire-time view angles. |
| D-34 | **Mid-range reference hardware** (O-07): 6-core Zen 2–class CPU, RTX 3060 / RX 6600–class GPU, 16 GB, Windows 11. | Every target in §9 is measured on it. |
| D-35 | **Working title Earthrise, and the title is not in code** (O-08). | Namespaces, projects and binaries use neutral names, so a rename is branding, not a refactor. |
| D-36 | **Long-term ceiling of 300–600 humans per continent server** (O-09): one multithreaded process, not a distributed one. | The MVP simulation stays single-threaded but must remain partitionable (§7.1). |
| D-37 | **No water in the MVP map** (O-10). | The concept art's rivers become dry canyons, or wait. |
| D-38 | **AGENTS.md updated, and R18–R29 adopted from this document** (O-11). | Code is reviewed against them from M0. §13 records where each comes from. |
| D-39 | **Grid movement:** a one-voxel ledge is stepped automatically, a two-voxel ledge needs a jump; doorways at least 2 voxels wide and 4 high (O-12). | Hills and rubble are walkable. A blueprint that breaks the clearance rule is a content bug. |
| D-40 | **Soldiers animate as rigid parts** (O-13): one low-poly piece per body segment, keyframed, no skinning. | No skinning code and cheap instancing; a toy-soldier look that suits the voxel world. |
| D-41 | **Faction colours are blue, red and amber** (O-08). | The three differ in lightness as well as hue, so red–green colour blindness does not merge them, and they are not PlanetSide 2's scheme. HUD and minimap back colour up with shape. |

## 4. The game, in one session

The player starts the client, presses Play, and the client launches the server with a map, a seed and a bot count. The deploy screen shows the lattice map with ownership, the player picks a class and a spawn point, and drops into a war that was already running and will keep running when they quit.

The **minute-to-minute loop** is: deploy, move to the front, fight over control points, die, get revived or respawn, fight again. The **fortification loop** runs inside it: engineers raise walls and bunkers on build slots, assaults blow holes through them, and both sides repair what is left. The **territory loop** runs above both: capturing a base extends the lattice, feeds the faction's build pool, and opens the next base to attack. Three factions means no front is ever only two-sided — whoever looks strongest gets attacked from two directions, which is the anti-steamroll dynamic PlanetSide 2 relies on.

There is no pause. The server is a separate process running a world that does not belong to the player, and the menu does not stop it. That is a direct consequence of D-01 and D-02, and it is deliberate.

## 5. World

### 5.1 Map and lattice

The map is about 2×2 km: three warpgates at the corners of a rough triangle, and seven capturable bases between them. Each faction has two bases near its warpgate, one facing each enemy, and a large neutral base sits in the centre. The layout is rotationally symmetric, so no faction starts with a better position.

Seven bases replaced the "3–5" first discussed, because fewer cannot give three factions a front against each other plus a contested middle; seven is the smallest symmetric layout that can (D-29).

| Node | Start owner | Size | Points | Lattice links |
|---|---|---|---|---|
| WG-1 | Faction 1 (permanent) | Warpgate | — | 1·2, 1·3 |
| 1·2 | Faction 1 | Medium | 2 | WG-1, 2·1, C |
| 1·3 | Faction 1 | Small | 1 | WG-1, 3·1 |
| WG-2 | Faction 2 (permanent) | Warpgate | — | 2·3, 2·1 |
| 2·3 | Faction 2 | Medium | 2 | WG-2, 3·2, C |
| 2·1 | Faction 2 | Small | 1 | WG-2, 1·2 |
| WG-3 | Faction 3 (permanent) | Warpgate | — | 3·1, 3·2 |
| 3·1 | Faction 3 | Medium | 2 | WG-3, 1·3, C |
| 3·2 | Faction 3 | Small | 1 | WG-3, 2·3 |
| C | Neutral | Large | 3 | 1·2, 2·3, 3·1 |

Base `x·y` is faction *x*'s base on the front facing faction *y*. Adjacent bases sit 350–550 m apart; walking 500 m at sprint speed takes about 75 s, which is the cost of having no vehicles and is why spawn rules matter (D-28).

### 5.2 Terrain

Terrain is a heightfield of columns on the same 0.5 m grid as the voxels: 4,000 × 4,000 columns, each with a height in 0.5 m steps and a surface material. It renders as blocks — grass tops, earth and rock sides, vertical cliffs — and collides as blocks, so what the player sees is what they walk on. It is authored offline (procedural generation plus hand edits in a tool) and shipped as content; it is not generated at runtime and does not change during play.

There is no water in the MVP map, although the concept art has a river. Swimming, water rendering and water in navigation are deferred (D-37). Trees and rocks are static voxel props with simple collision and are not destructible.

### 5.3 Bases

A base is a bounded voxel volume — up to 160 × 160 × 48 m, or 320 × 320 × 96 voxels — that sits on the terrain and overrides it inside its bounds. Every base has three layers:

The **hardened core** is a spawn room, the control-point plinths, the loadout terminals, and the anchors that hold the structure up. Hardened voxels cannot be destroyed by anything (D-23). This keeps the capture loop intact.

The **fortification shell** is the authored walls, buildings, stairs, ramps and cover around the core. It is destructible and is the blueprint that restoration and repair return to.

The **build slots** are marked footprints where engineers may place modules (§6.7). A slot is empty in the blueprint.

Buildings have interiors with multiple floors. Fights inside and around buildings are where PlanetSide 2's infantry play happens, and they are what makes breaching worth doing.

Warpgates are hardened throughout, contain only spawn and terminals, and cannot be attacked.

### 5.4 Voxels and materials

Voxels are 0.5 m cubes. Each holds one material from a palette, and the material decides how it looks (colour, emissive glow, roughness) and how hard it is to destroy:

| Resistance | Destroyed by | Typical use |
|---|---|---|
| Light | Grenades and breach charges | Sandbags, light cover, module panels |
| Heavy | Breach charges only | Base walls, building shells |
| Hardened | Nothing | Spawn rooms, point plinths, terminals, warpgates, anchors |

There is no per-voxel damage state in the MVP: an explosion either removes a voxel or it does not, by comparing the blast strength at that voxel with its material's resistance. Repeated grenades do not chip a heavy wall. That keeps edits stateless, deterministic and cheap to replicate (§7.4).

## 6. Gameplay

### 6.1 Factions

Three factions, symmetric, coloured **blue, red and amber** (D-41). Names are open (O-14). Faction colour is the primary readability tool in a voxel world and is used on characters, emissive strips, territory and the minimap. The HUD and minimap also give each faction its own shape, so colour is never the only cue.

### 6.2 Soldiers

Every soldier has **shields** that regenerate after a few seconds without damage and **health** that does not regenerate — only a medic restores it. Damage hits shields first. This is PlanetSide 2's model and is what makes the medic class necessary rather than optional.

A soldier killed is **downed**, not gone: they can be revived for a window (proposed 30 s) or choose to respawn at any time. A revived soldier returns with partial health.

All numbers — shield, health, regeneration delay and rate, revive window, damage — are tuning data, not code (R21). The design target is a time-to-kill of about 0.5–0.9 s for a primary weapon at medium range with accurate fire.

### 6.3 Spawning and the deploy screen

The deploy screen shows the lattice, ownership, capture progress and the faction's build pool. The player picks a class and a spawn point: the warpgate, or an owned base. Respawn takes a fixed time (proposed 10 s), which the director may shift within bounds (§6.10).

**A base under attack** keeps its spawn for the defenders until the attackers hold every one of its control points (D-28). Losing the spawn is the decisive moment of an attack, and a full sweep of the points is what an attacking squad works toward.

### 6.4 Capture

A base can be attacked only by a faction that owns a base linked to it on the lattice. A control point flips to a faction that stands on it uncontested; flip time falls as more soldiers of that faction stand on it, down to a floor (proposed 12 s with one soldier, 6 s with four or more). Any enemy presence on the point freezes it.

Once an attacking faction holds a majority of a base's points, the base capture timer runs (proposed: small 90 s, medium 150 s, large 240 s), at double speed if they hold every point. If the owner regains the majority, the timer runs backwards. Progress belongs to one attacker at a time: if a second attacking faction takes the majority, the first's progress drains to zero before the second's starts. That is the three-way rule in its simplest form. When the timer completes, the base and its points change owner, and every module in the base crumbles (D-32).

### 6.5 Classes

Three classes (D-30), each with one primary weapon, a shared sidearm and a shared frag grenade:

**Assault** — carbine or assault rifle, and **breach charges**: placed explosives that destroy light and heavy voxels. Ability: a short overshield. Assault is the class that opens walls.

**Medic** — assault rifle, a **heal tool** that restores allies' health, and **revive** for downed allies. Ability: an area heal.

**Engineer** — carbine, an **ammo pack** that resupplies nearby allies, and the **build tool**: place modules on build slots and repair damaged voxels back to blueprint. Engineer is the class that builds.

Jump jets were left out on purpose: vertical movement over walls is a natural fit for a voxel world, but it multiplies navigation cost for bots. They are the first candidate after the MVP.

Class is chosen on the deploy screen or changed at a loadout terminal.

### 6.6 Movement

Walk, sprint, crouch, jump. No prone — PlanetSide 2 has none. The 0.5 m voxel grid sets the shape of movement (D-39):

A single-voxel ledge (0.5 m) is **stepped up automatically**, with the camera smoothed over the step, so hillsides and rubble are walkable without jumping. A two-voxel ledge (1.0 m) needs a jump; the jump apex is proposed at about 1.1 m. Anything higher needs stairs, a ramp or a breach.

The soldier's world collision is an axis-aligned box: about 0.7 m wide and 1.8 m tall, 1.3 m crouched. That gives the blueprint clearance rule that level and module authoring must follow: **doorways at least 2 voxels wide and 4 high; crouch gaps at least 3 high.** A blueprint that violates it is a content bug.

There is no fall damage in the MVP.

### 6.7 Construction

Construction happens only inside bases the faction owns, on build slots, using modules from a small library. The proposed MVP library: wall segment, bunker (cover with firing slits), ramp/stairs, and watchtower. A module is a blueprint like the base's own shell.

Placing a module spends resource from the faction pool and creates a frame. The frame fills in voxel by voxel while an engineer works on it with the build tool, and the module is fully solid only when the last voxel is in. A module that has started building can be shot at and blown up like any fortification. When a base changes owner, every module in it crumbles and its slots are empty again (D-32): the new owner starts from the bare shell, and fortifying a base is a bet that it will hold.

**Repair** restores destroyed shell or module voxels back to blueprint, faster than restoration and at a resource cost per voxel. **Restoration** is the free, slow process that runs on its own: a base that has seen no enemy damage or presence for a set time (proposed 30 s) slowly fills its missing blueprint voxels. Both work **bottom-up in a fixed order**, so that a restored voxel always has support (D-26), and neither fills a voxel that a soldier is standing in; that voxel waits.

**Resource:** each owned base adds a fixed income per minute to its faction's pool, up to a cap. Bigger bases pay more. Losing territory starves a faction's building, which is the feedback loop between territory and construction (D-25). The director may adjust income within bounds.

### 6.8 Destruction

Frag grenades destroy light voxels within a small radius. Breach charges destroy light and heavy voxels within a larger one. Nothing else edits the world: bullets, melee and future weapons not marked as explosive leave voxels alone (D-23). Every explosion is an operation applied to the grid in exact integer arithmetic (§7.4).

After every edit, a **support check** finds voxels that are no longer connected to an anchor — terrain, a base foundation layer, or a hardened voxel. Those voxels **crumble**: they are removed with a burst of debris and a small amount of damage to soldiers directly beneath. The check is bounded; if it would visit more than a fixed number of voxels it treats the piece as supported. That can leave an occasional large overhang standing, which is accepted in exchange for a bounded cost.

Debris is cosmetic and client-side only. It does not collide with soldiers, is not replicated and has no effect on gameplay.

### 6.9 Weapons and hits

Weapons fire projectiles with velocity and drop, not instant rays. Spread, bloom, recoil and aiming down sights are tuning data. Head hits deal extra damage.

The server decides every hit (D-10). It simulates each projectile and tests it against hitboxes rewound to the moment the shooter saw them, capped at a maximum rewind (proposed 250 ms) to bound the advantage a high-latency shooter gets. The world is **not** rewound: projectiles collide with the voxels as they are now. A shot fired at a target through a gap that has just been sealed hits the wall; a shot through a hole just blown open goes through. This is a fairness trade-off and is written down so it is not rediscovered as a bug.

Hitboxes are boxes per body part. They are computed from replicated state — position, stance, yaw, pitch — not from client animation, so the server never has to run animation to know where a body is (R24).

Friendly fire comes from explosives only (D-31): grenades and breach charges hurt allies, while bullets pass through them. It is a data flag. Before throwing or placing an explosive, a bot checks that the blast will not catch allies.

### 6.10 Population director

The director exists to keep an endless three-faction war from locking (D-16). Its levers and proposed bounds:

| Lever | Nominal | Bounds |
|---|---|---|
| Soldiers per faction | 33 (faction 1: 32 bots + the human) | 27–39 |
| Respawn time | 10 s | 7–13 s |
| Build income | 1.0× | 0.75–1.25× |

It reads territory share and its trend, and moves the underdog toward advantage only within those bounds. It never touches combat numbers — damage, health, accuracy — because a war whose fights are secretly rigged is not worth winning. It treats the human's faction exactly like the others. Every adjustment is logged as an event so its effect can be measured.

### 6.11 Bots

A bot is a soldier with no socket. It produces the same command stream a client sends — move, look, fire, use ability, build, repair — and that command stream goes through exactly the code a human's does. Nothing in the AI writes simulation state directly (R19), which means every bot is also a test of the movement, weapon and construction code.

Bots are organised in three layers:

The **faction commander** (about once a second) owns strategy. It reads the lattice, ownership, capture timers, the build pool and its own soldiers' positions, and chooses where to attack, what to defend, and which modules to build where. Modules crumble when a base falls (D-32), so it weighs a module's cost against how likely the base is to hold. It assigns squads to objectives.

The **squad** (six to eight bots) carries out an objective. It routes across the map, chooses which control point or approach to take, decides whether to breach a wall or go around, and keeps engineers and medics doing their jobs.

The **individual** (decisions about ten times a second; aim and movement every tick) perceives, picks targets, moves, takes cover, shoots, reloads, throws grenades, revives, resupplies, builds, repairs, and places breach charges when its squad has decided to breach. Action selection is utility-scored.

Bots perceive; they do not read the world. Decision code sees only what the bot has seen or heard — enemies in a view cone with clear line of sight, gunfire and explosions within earshot — and never global truth (R20). The exception is information every player has on the map: lattice, ownership and capture progress.

Aim is modelled on a human: reaction time, an error that closes while tracking, and a turn-rate limit. Each bot has a skill value drawn from a distribution so the population is mixed.

**Breaching** is part of navigation, not a special behaviour. Navigation knows which walls are destructible and how thick they are, and represents a breachable wall as a path edge with a cost: time to set and fire the charge, plus exposure. When a squad's best route goes through such an edge, an assault bot is tasked with blowing it, and the squad waits, then pushes through. When engineers repair the wall, the edge closes again.

**Building** is driven from the commander down. The commander decides which bases need fortifying — front-line bases first — and which modules to buy with the pool. Each base keeps a task list (slots to build, damaged areas to repair, ordered by distance from points and spawn exits), and engineer bots near or assigned to that base work through it.

### 6.12 HUD and front end

Crosshair with hit markers, shield and health, ammunition, ability cooldown, damage-direction indicator, minimap with lattice and ownership, capture progress for the base you are in, a kill feed, and a build-tool overlay showing slots, blueprints and pool balance for engineers. The deploy screen (§6.3) and a settings menu: mouse sensitivity, field of view, volume, display mode and resolution, key bindings.

Keyboard and mouse only. Mouse input is raw.

### 6.13 Seams left open

Two things are not in the MVP but would be expensive to retrofit, so the design reserves room for them. **Possession:** a soldier's input drives an entity, and the code does not assume that entity is always a soldier — that is where vehicles and manned turrets attach later. **More humans:** the server never assumes one client, the protocol carries a client id everywhere it would need one, and the second-client test in §9 keeps that honest.

## 7. Technical design

### 7.1 Processes

The client starts the server as a child process, passing the map, seed, bot count and network settings on the command line. The server binds a UDP port and tells the client which one; the client connects over loopback. The child is placed in a job object that kills it if the client exits, so a crashed client never leaves an orphaned server. If the server crashes, the client reports it and offers a restart.

The server runs with no window and no graphics. It can also be started on its own as a dedicated server, from the same binary, from the first milestone.

Code divides into: an engine core used by both processes (platform, maths, containers, logging, tasks); networking used by both; the shared game simulation — movement, collision, weapons, voxel operations, capture rules — compiled once and linked into both (R25); the renderer, audio and input, client only; AI and the director, server only. The layout of projects is ADR-001's to decide, within AGENTS.md §2 and R9. The game's title appears in none of it (D-35).

**Headroom for scale.** The long-term target is 300–600 humans on one continent server (D-36): one process, multithreaded, not distributed. The MVP simulation runs on one thread, but nothing in it may be built so that it cannot be split later. That rules out any loop over all pairs of soldiers. Spatial queries go through a partition rather than a scan, relevance for a client costs in proportion to what is near it, and every system's per-tick work can be divided by region. A design that only works because there are a hundred soldiers is a defect, even if the MVP never notices it.

### 7.2 Simulation

The server simulates at a fixed tick rate. It starts at **30 Hz**; the final rate is chosen by measurement at M3 and M6 and recorded in ADR-004 (D-33). The client renders at the display's rate, predicts its own soldier with the shared movement code, and interpolates everyone else between snapshots. Commands carry the exact view angles at the moment of firing, so aim precision does not depend on the tick rate.

**Time is ticks.** Simulation code reads time as a tick count; wall-clock time is converted at one place (R22).

**Space is metres**, origin at the map centre. At ±1 km a `float` resolves about 0.06 mm, and at ±4 km — a full PlanetSide-scale continent — about 0.24 mm, so no double precision or origin rebasing is needed now or later. The voxel grid uses integer coordinates. The axis convention is ADR work (§11).

The simulation as a whole is **not deterministic** and does not need to be: the server sends state and the client corrects towards it. The exception is voxel operations, which must produce identical results on the client and the server because they are replicated as operations, not as results (§7.4). That is the "deterministic core" AGENTS.md §5 says must be recorded as an ADR, and its scope is exactly the voxel grid: integer coordinates, integer or fixed-point blast arithmetic, fixed traversal orders, no `float` anywhere in it. AI uses a fixed PRNG with a recorded seed, so a soak run can be reproduced closely enough to debug, though not bit-exactly.

### 7.3 World, collision and physics

One query model serves both terrain and bases: *is this voxel solid?* Inside a base's bounds the base volume answers; outside, the terrain column does. On top of that, all hand-written:

- **Movement:** the soldier's box swept through the grid, axis by axis, with step-up and slide. Shared by client prediction and server.
- **Rays:** grid traversal (DDA) for line of sight and for projectiles against the world, then a test against hitboxes.
- **Explosions:** the set of voxels within a radius, used for both voxel removal and splash damage.
- **Support:** a bounded flood-fill after each edit (§6.8).

There are no rigid bodies in the MVP. Grenades are simple ballistic objects that bounce off the grid. Debris is a client-side particle effect.

Voxel storage is chunked in 32³ chunks with a material palette; a chunk of uniform air or uniform solid is stored as one value. A full-size base is 10 × 10 × 3 = 300 chunks; at one byte per voxel, a chunk that is not uniform is 32 KiB. Even if a third of every base were non-uniform, ten volumes (seven bases, three warpgates) come to about 32 MiB before compression. The terrain is 4,000 × 4,000 columns with one byte of height and one of material, 32 MB. These are calculations from the stated sizes, not measurements.

### 7.4 Networking

**Transport:** UDP over Winsock, with an unreliable sequenced channel for snapshots and commands and a reliable ordered channel for events. Packets stay under 1,200 bytes. The handshake checks a protocol version hash, so a client and server from different builds refuse each other cleanly.

**Network conditioner:** built into the transport from the first milestone, configurable per direction — latency, jitter, loss, duplication, reordering. Loopback is a perfect network, and a netcode tested only on loopback is untested.

**Client to server:** one command per tick, each packet also carrying the last few commands so that a lost packet loses nothing.

**Server to client:** snapshots delta-compressed against the last snapshot the client acknowledged. Every entity within a relevance radius is sent, prioritised by distance and refreshed less often as it gets further away. Line of sight is not used to filter, to avoid enemies popping in. The relevance radius must be at least the render distance, and the render distance at least the longest engagement range (R26): if you can be shot by it, you can see it.

**Voxel edits are replicated as operations.** The server applies an operation — an explosion at a voxel position with a radius and strength, a module placement, a batch of repair or restoration steps — and sends the operation, stamped with its tick, on the reliable ordered channel. The client applies the same operation to its own copy and arrives at the same voxels. An explosion that removes four hundred voxels costs about sixteen bytes. Crumbling is not sent at all: both sides compute it from the same edit. Clients send a hash of each chunk periodically; on a mismatch the server sends that chunk in full and logs it as a determinism bug. A client joining late receives every chunk that differs from its blueprint.

The client does not predict voxel edits; it waits for the server's operation, a round trip later. It may show the explosion's effect at once.

**Estimated budget:** 100 entities at about 10 bytes per delta update, averaging 15 updates a second across the distance tiers, is about 15 KB/s, or 120 kbit/s, plus headers and voxel operations. The target is 256 kbit/s at the 95th percentile per client. This is an estimate to be replaced by measurement.

**Lag compensation:** the server keeps a second of hitbox history (30 ticks × 100 soldiers at 30 Hz) and rewinds up to the cap in §6.9. Bots are on the server and have no latency to compensate.

### 7.5 Navigation

The navmesh baked offline that a static world would allow is ruled out, because walls disappear and appear at runtime. Navigation is built from the voxel grid and updated as it changes:

Each column holds a list of **walkable spans**: places with solid ground beneath and at least four voxels (2 m) of air above, or three for crouch-only spaces. Spans link to neighbours they can step up or down to, jump up to, or drop to. That is the fine graph.

Above it sits a **hierarchical graph**: one cluster per 32 × 32 columns, with portals where clusters connect. Long paths are planned on the cluster graph and refined locally. An edit rebuilds only the spans and portals of the chunks it touched, time-sliced so the server's budget holds; the target is that navigation reflects an edit within two ticks.

**Breach edges** cross destructible walls up to a set thickness, carrying the cost described in §6.11. They are found and updated by the same chunk rebuild.

Above both is the **lattice**: the commander reasons base to base and only asks for paths between them.

Bots avoid each other with local steering, not by planning around one another.

### 7.6 Rendering

Direct3D 12 on the inbox runtime of Windows 11 (D-12). The renderer's shape — passes, targets, anti-aliasing, scaling, text — is ADR work under AGENTS.md R12; the design states what it has to show.

**Voxels:** each chunk is turned into a mesh by merging coplanar faces of the same material, with per-vertex ambient occlusion; that look is most of what a voxel style is. When an operation changes a chunk, its mesh is rebuilt on a worker thread within a per-frame budget, and the change is visible within two frames of the operation arriving. Distant chunks use meshes built from a voxel grid downsampled 2× and 4×.

**Terrain:** the column field meshed by chunk in the same way, with coarser meshes in the distance, out to the full 2 km.

**Lighting:** one sun with shadows, sky, distance fog, per-vertex occlusion, emissive materials with bloom — the glowing faction strips in the concept art carry identity and readability.

**Characters:** a hundred soldiers on screen in the worst case, drawn instanced as low-poly rigid parts (D-40), with lower-detail versions in the distance. Rigid parts instance cheaply: a soldier is a dozen or so part instances, each just a transform, and there is no skinning pass. They are not voxels, but they are shaded to sit in the voxel world: flat or near-flat colour and a strong faction colour. A soldier must stay visible at least out to the longest engagement range (R26).

**Shaders** are compiled into the executable (AGENTS.md §2).

### 7.7 Audio

XAudio2 with 3D positioning: gunfire, footsteps, explosions, crumbling, construction, capture alarms. Footsteps and gunfire direction are gameplay information in a shooter, not decoration, so positional audio is MVP scope, not polish.

### 7.8 Content pipeline

There are two content paths, and the game reads only formats it defines for each; the converters live in `Tools/` and may use third-party code (D-03).

**World content** — base blueprints, modules and voxel-styled props such as trees and rocks — is authored in a voxel editor and converted into blueprint and prop formats. Terrain is generated by a tool and edited by hand.

**Moving things** — soldiers, weapons, and later vehicles — are low-poly meshes authored in a 3D modelling tool and converted into mesh and animation formats. Soldiers animate as rigid parts: one low-poly piece per body segment, driven by keyframed transforms, with no skinning (D-40). The segments match the hitbox layout, but hitboxes are still computed from replicated state. Animation is client-only; the server never evaluates it (R24).

The map file — bases, points, slots, lattice, spawns — and all tuning data are text, readable at startup and reloadable while running in development builds. Which editor and which file formats is ADR work (§11). Third-party content needs the owner's approval and carries its licence with it (AGENTS.md R14).

### 7.9 Debugging and observability

There is no Dear ImGui (R14), so the MVP builds its own minimal immediate-mode debug UI: a console with variables, a network graph, server tick-time breakdown, and world overlays for navigation spans, clusters, breach edges, bot perception and decisions, chunk boundaries and remesh activity. Every gameplay event — kill, revive, capture, edit, director adjustment — is logged with tick, instigator, target, factions and position (R27), which feeds the kill feed, the metrics in §9 and debugging alike.

### 7.10 Testing

Unit tests for the shared simulation — movement, collision, voxel operations, capture rules — run in CI per AGENTS.md §3. Voxel determinism has its own tests: the same sequence of operations, applied from the same blueprint, yields identical chunk hashes in every configuration CI builds.

The **headless soak** is the MVP's main test: the server alone with 99 bots, no client, for hours, reporting the flow and performance metrics of §9. A second soak attaches a **verification client** that does nothing but receive and compare chunk hashes, which is how determinism is checked at scale.

## 8. Out of scope for the MVP

Vehicles of any kind, and aircraft. Mobile spawns. Water. Destructible terrain. Free-form building and resource harvesting. Classes beyond three, jump jets. Asymmetric factions. Progression, unlocks and any saved state other than settings. Matchmaking, server browser, accounts, anti-cheat and encryption. Controller support. Voice and text chat. Day–night and weather. Squad or platoon command by the player. A second continent.

## 9. Definition of done

These are targets. They are measured on the reference hardware (D-34: 6-core Zen 2–class CPU, RTX 3060 / RX 6600–class GPU, 16 GB, Windows 11) and recorded with how they were measured.

**Play:** the player can deploy, change class, fight, die, be revived, respawn, capture and defend bases, build modules, repair, and breach — and every one of those is also done by bots without human input.

**Flow** — over a two-hour unattended soak:

- No crash, no server stall.
- At least six base captures per hour.
- No faction holds more than 70% of the capturable bases for more than fifteen minutes in a row.
- Every faction captures at least one base per hour.
- Every class uses its ability, and bots carry out builds, repairs and breaches every hour.
- Less than 1% of bot-time spent stuck (no progress toward a goal for more than 10 s).

**Server:** tick time at the 99th percentile no more than half the tick budget (16.7 ms at 30 Hz) on one simulation thread with 100 soldiers, during the heaviest fights the soak produces. Navigation reflects an edit within two ticks.

**Client:** at 1440p in the worst base fight, an average of at least 120 frames a second and a 99th-percentile frame time of no more than 12 ms. A voxel edit is visible within two frames of its operation arriving.

**Network:** under the conditioner at 100 ms round trip, ±20 ms jitter and 2% loss, the game is playable: own-movement corrections over 5 cm happen less than once a minute when not touching other soldiers; at least 98% of shots that hit on the shooter's screen are confirmed by the server; downstream stays under 256 kbit/s at the 95th percentile. In a soak with a verification client, the number of chunk hash mismatches is zero.

**Net-ready:** a second client on another machine joins a running server from the command line and plays, with no code change.

## 10. Risks

In order of how badly each could sink the MVP.

**Bot AI on a world that changes.** The human's whole experience depends on bots that fight, build, repair and breach believably, on navigation that is rebuilt under them. This is both the hardest and the most important system in the MVP. Mitigated by building navigation and its debug view before any combat AI, by the stuck metric, and by the cut order below.

**Hand-writing everything (R14).** Collision, navigation, netcode, UI and text, each from scratch. The voxel decision helps here — box-against-grid collision and grid raycasts are far simpler than triangle-mesh collision — but the total is still large.

**Voxel determinism.** One floating-point operation in the voxel path, one unordered container, or one difference between Debug and Release code generation, and the client's world quietly drifts from the server's. Mitigated by keeping the deterministic scope small and integer-only, testing it in CI, and checking chunk hashes in every soak.

**Three-faction flow.** The war may still lock up or steamroll despite the director. The flow metrics make that visible within an hour of soak rather than after weeks of play.

**Performance in the worst fight.** A hundred soldiers, explosions, remeshing and navigation rebuilds all land in the same seconds in the same base. The budgets in §9 are checked in soak from the milestone where each system lands, not at the end.

**Loopback hiding network problems.** Mitigated by the conditioner from the first milestone.

**The inbox D3D12 runtime.** Without the Agility SDK, which features and shader models are available depends on the Windows build. Verified on day one and recorded.

**Time to first fun.** One developer, a long road. The milestones put "shoot bots in a voxel base" before any of the territory layer.

**If behind, cut in this order:** breach planning by bots (bots build and repair only); then modules (repair and restoration only); then three-point bases (two at most); then distant-chunk detail levels (shorter view of distant bases). The three pillars survive every one of those cuts.

## 11. Engineering decisions this design forces

Each is an ADR to be written when the code that implements it is, per AGENTS.md §6. They are listed here so they are not discovered late.

| ADR | Decision | Notes |
|---|---|---|
| ADR-001 | Solution and project layout | Engine core / net / shared simulation / renderer / client / server / tests; server must not link graphics |
| ADR-002 | Server process lifecycle | Child process, port hand-off, job object, crash handling |
| ADR-003 | Transport and reliability | Channels, packet format, conditioner |
| ADR-004 | Tick rate | Starts at 30 Hz, chosen by measurement (D-33) |
| ADR-005 | Snapshots, deltas and relevance | Tiers, priorities, budget |
| ADR-006 | Coordinates and units | Axis convention and handedness; metres; integer voxel grid |
| ADR-007 | Deterministic voxel core | Scope, integer rules, PRNG, hash protocol (AGENTS.md §5) |
| ADR-008 | Voxel storage and chunking | Chunk size, palette, compression |
| ADR-009 | D3D12 baseline on the inbox runtime | What Windows 11 exposes without the Agility SDK, verified |
| ADR-010 | Renderer shape | Passes, anti-aliasing, scaling, window modes (R12) |
| ADR-011 | Text and debug UI | No third-party; baked bitmap font or DirectWrite interop |
| ADR-012 | Content formats and pipeline | Voxel editor and modelling tool; blueprint, prop, mesh, rigid-part animation (D-40), map and tuning formats |
| ADR-013 | Navigation representation | Spans, clusters, breach edges, update budget |
| ADR-014 | Threading | Simulation thread, workers for meshing, navigation and perception; must leave room for 300–600 humans (D-36) |

## 12. Open questions

Each carries a recommendation. Answered questions move to §3 as decisions and keep their number here for reference: O-01 to O-13 were answered as D-28 to D-41.

| # | Question | Recommendation |
|---|---|---|
| O-14 | Faction names and the game's fiction | Not needed before M6. The working names are Faction 1–3 until then. |
| O-15 | The module library | Wall segment, bunker, ramp/stairs, watchtower (§6.7). Confirm before M7. |

## 13. Conformance rules from this design

AGENTS.md reserves R18 onward for rules whose source is a design document. These twelve were adopted into AGENTS.md as R18–R29 (D-38). AGENTS.md carries the rules; this table records where each one comes from. A new rule this document gives rise to is proposed here as a candidate `C-nn`, and takes the next free R number when the owner adopts it.

| # | Rule | Source |
|---|---|---|
| R18 | **The server decides.** Client code never decides a hit, damage, death, capture or voxel edit; it predicts and displays, and the server confirms. | D-10, §7.4 |
| R19 | **A bot is a client without a socket.** Bots act only through the player command interface; AI never writes simulation state. | §6.11 |
| R20 | **Bots perceive; they do not read.** AI decision code reads the world only through the perception system, apart from information on every player's map. | §6.11 |
| R21 | **Tuning lives in data.** Damage, speeds, timers, costs and director bounds are data files, never code constants. | §6.2 |
| R22 | **The tick is the clock.** Simulation reads time in ticks; wall time is converted at one place. | §7.2 |
| R23 | **Nothing is two-sided.** Faction relations go by faction id; no `bool` enemy flags and no faction count in code. | D-05 |
| R24 | **Hitboxes come from replicated state,** never from client animation. | §6.9 |
| R25 | **Shared simulation is shared code.** Movement, collision, weapons, voxel operations and capture rules are compiled once and linked into both client and server. | §7.1 |
| R26 | **If it can shoot you, you can see it.** Relevance radius ≥ render distance ≥ longest engagement range. | §7.4 |
| R27 | **Events carry their context:** tick, instigator, target, factions and position. | §7.9 |
| R28 | **One door into the voxel grid.** Every voxel change is an operation through one pipeline that applies, logs and replicates it; nothing writes chunk storage directly. | §7.4 |
| R29 | **The voxel core is integer.** No `float` in voxel operations, fixed traversal order, no unordered iteration that reaches a result. | §7.2, AGENTS.md §5 |

## 14. Milestones

Each milestone ends with something that runs and can be looked at, per AGENTS.md §3.

**M0 — Foundations.** Solution and projects (ADR-001); the three checkers AGENTS.md asks for; CI green; a window with a cleared D3D12 swap chain; logging; the inbox-runtime survey (ADR-009).

**M1 — Two processes.** The client launches the server; transport with the conditioner; handshake; a box moved by the player, predicted and reconciled, under 100 ms of simulated latency.

**M2 — Voxel world.** Terrain columns and one base volume, meshed with occlusion; box-against-grid movement shared by client and server; first-person walking, stepping and jumping over the network.

**M3 — Combat.** Projectile weapons, hitboxes, lag compensation, shields and health, downing and respawn, deploy screen, hit feedback.

**M4 — Destruction.** Materials, grenades, breach charges, crumbling, operation replication with chunk hashes, remeshing within budget, debris.

**M5 — Bots that move and fight.** Runtime navigation with incremental rebuild and its debug view; perception, aim, individual combat; thirty bots in one base.

**M6 — Territory.** Seven bases, points, lattice, capture, spawning, warpgates; commander and squads; the director; the full hundred-soldier soak with flow metrics.

**M7 — Build, classes, breach.** Faction pool, modules, repair, restoration; the three classes' kits; bots building, repairing and breaching.

**M8 — Acceptance.** HUD, audio, art pass, performance work until §9 holds on the reference hardware.

## 15. Glossary

**Lattice** — the graph of links between bases; a base can only be attacked along a link. **Warpgate** — a faction's permanent, uncapturable home base. **Control point** — a spot in a base that a faction holds by standing on it. **Blueprint** — the intended voxel shape of a base shell or module, which repair and restoration restore. **Build slot** — a footprint in a base where a module may be placed. **Breach edge** — a navigation link through a destructible wall, taken by blowing it. **Hardened** — a voxel material nothing can destroy. **Operation** — a voxel edit as sent over the network: what happened, where, at what tick. **Director** — the server system that keeps the three-faction war from locking.
