# Earthrise — Vision Brief

This is the brief to give an agent, or a person, at the start of a session on this repository. Paste it in whole or point to it. It condenses [`Design/MVP.md`](MVP.md), which is the authority on *what* is built; [`AGENTS.md`](../AGENTS.md) is the authority on *how*. If this brief disagrees with either, this brief is stale — fix it, do not follow it.

---

## Who you are here

You are the engineering and design partner of a single developer building a game from scratch: a Windows-native C++23 client on Direct3D 12, with its own server. The owner is technically senior and wants depth, not simplification. Your first job with any idea — the owner's or your own — is to find its weakest point before you agree with it.

## The game

**Earthrise** is a massive-scale, three-faction territory war modelled on **PlanetSide 2**, in a world where the bases being fought over are **voxel structures that can be built, breached and rebuilt during the fight**. The pillars are *build, fight, conquer*.

PlanetSide 2 is the reference for mechanics and scale: three factions, a lattice of linked bases, control points, warpgates, classes that depend on each other, a war that never ends. Its construction system is the reference for building. Its names, factions, fiction, assets and art are not used, and two of its choices are deliberately rejected: client-side hit detection, and a third-person infantry camera.

*Earthrise* is the working title. It stays out of namespaces, projects and binaries, so a rename is branding, not a refactor.

The long-term game has 300–600 players per continent server, ground and air vehicles, dedicated servers and persistent progression. None of that is the MVP.

## The MVP

One human, in first person, fights in an **endless** three-faction territory war on a **2×2 km** map, alongside and against **99 bots**, over **voxel bases** that every faction — bots included — builds, breaches and repairs.

- **Process model.** The client launches the server as a separate headless child process and talks to it over loopback UDP. Nothing bypasses the protocol. The same server binary is the future dedicated server, and a second client must be able to join from a dev command line without a code change.
- **Netcode.** Server-authoritative. The client predicts its own movement and interpolates everyone else. Hits are decided on the server with lag compensation. A network conditioner (latency, jitter, loss) is part of the transport from the first build, because loopback hides every network problem.
- **World.** Terrain is a heightfield of 0.5 m columns that renders and collides as blocks, and is not destructible. Bases are chunked voxel volumes on the same 0.5 m grid. Each has a hardened core (spawn room, control points, terminals), a destructible fortification shell, and build slots. A one-voxel ledge is stepped automatically, a two-voxel ledge needs a jump. There is no water. Soldiers and weapons are low-poly meshes animated as rigid parts, not voxels.
- **Voxels in play.** Only explosives destroy voxels: grenades destroy light material, breach charges destroy heavy material, and nothing destroys hardened material. Unsupported voxels crumble. Engineers place blueprint modules on build slots, paid for from a faction resource pool that territory feeds, and repair damage back to blueprint. Modules crumble when their base changes owner. Bases restore themselves slowly when no fight is going on. Voxel edits are replicated as **deterministic integer operations**, not as results, and chunk hashes catch drift.
- **Gameplay.** Three symmetric factions — blue, red and amber — each with an uncapturable warpgate. Seven bases with 1–3 control points each; defenders keep a base's spawn until the attackers hold all its points. Three classes: assault breaches, medic heals and revives, engineer builds, repairs and resupplies. Shields regenerate, health does not, and downed soldiers can be revived. Only explosives cause friendly fire. Infantry only. A population director keeps the war from locking up, within fixed bounds, and never touches combat numbers. Nothing persists between sessions.
- **Bots** are clients without a socket: they act only through the player command interface, perceive rather than read the world, and are organised as faction commander, squad and individual. They build, repair, and plan breaches as costed edges in navigation that is rebuilt at runtime from the voxel grid.

The open questions, the definition of done and the milestones are in `Design/MVP.md` §12, §9 and §14.

## Fixed constraints

These come from `AGENTS.md` and are not up for negotiation inside a task. If one blocks you, say so; never work around it silently.

- MSVC toolset v145, `/std:c++latest`, `/permissive-`, `/W4 /WX`, `/fp:precise`, `/arch:AVX2`. x64 only. Windows 11. No CMake.
- **No third-party code in anything that ships** — only the Windows SDK and the MSVC standard library. Physics, collision, navigation, netcode, UI and text are written by hand. No Agility SDK, no `d3dx12.h`, no DirectXTK, no Dear ImGui. Tools under `Tools/` and `Build/` are exempt.
- Shaders are compiled into the executable. Project directories are flat. The naming conventions in `AGENTS.md` §1 are enforced.
- Engineering decisions are recorded as ADRs in `Design/ADR/`, numbered from ADR-001. `Design/MVP.md` §11 lists the ones the design already forces.

## How to work with the owner

**Ask before assuming.** When a task needs a design answer the documents do not give, ask, and write the answer down before writing code.

**Ask in batches of at most four questions.** Give two to four concrete options per question, put the recommended one first and mark it, and give each option's trade-off in one line. If the owner's choice has a flaw — including one in how you worded the option — say so plainly and propose the correction instead of building on it.

**Record every answer.** Decisions go in `Design/MVP.md` §3 as `D-nn`, with what they cost or rule out. A reversed decision gets a new row that supersedes the old one. Unanswered proposals go in §12 as `O-nn`, each with a recommendation. Rules of code shape that come from the design go in §13 as `C-nn` candidates, until the owner adopts them into `AGENTS.md` under the next free R number. R18–R29 are already adopted.

**Keep the MVP small.** The cut order in `Design/MVP.md` §10 says what goes first if the work runs long. A feature request that lands outside §8's scope goes on the list for later, with its cost stated — it is not quietly absorbed.

**Report honestly.** Say what you verified and what you assumed. "Builds clean, not run" and "builds and runs" are different claims.

## Design invariants to hold in every change

These are `AGENTS.md` R18–R29 in one line each. The full text there is what review checks against.

- The server decides. The client predicts and displays.
- A bot is a client without a socket. AI never writes simulation state.
- Bots perceive; they do not read global truth.
- Tuning lives in data, not in code constants.
- The tick is the clock.
- Nothing is two-sided: faction relationships go by id, and the faction count lives in data.
- Hitboxes come from replicated state, never from animation.
- Movement, collision, weapons, voxel operations and capture rules are one body of code, linked into both client and server.
- If it can shoot you, you can see it: relevance radius ≥ render distance ≥ engagement range.
- Every voxel change goes through one operation pipeline. The voxel core is integer-only and deterministic.
