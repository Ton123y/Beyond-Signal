# Beyond Signal — Implementation Plan

## Product summary
A professional, multipage website about NASA with three interactive pillars:
1. **NASA knowledge hub** — agency history, the NASA insignia ("meatball", "worm" and seal) story, field centers, landmark missions timeline, and science directorates.
2. **Mission Academy quiz** — 5 levels (Foundations → Mission Expert), points per correct answer (more for harder levels, bonus for streaks, penalty for using hints), a hint for every question, explanations after each answer, and score submission to a public leaderboard dashboard.
3. **Mission Lab (3D game)** — design, manage and simulate a complete space mission based on the NASA Space Apps challenge brief: objective → requirements → spacecraft bus + instruments → launch vehicle + orbit → simulation with random events → scored mission report and post-mission analysis. Single player or **team mode (up to 4 players, roles on one device)**.
4. **Scenario Studio** — people design their own mission games (objective, budget/mass/power caps, required science, allowed orbits, random-event deck, difficulty) which are saved and playable by anyone in the Mission Lab. Includes step-by-step **tutorials**.

Background everywhere: a fixed **Three.js scene of moving planets** whose composition changes with scroll (Earth & Moon → Mars → Jupiter → Saturn & rings → deep-space nebula), reacting to the current page.

## Stack and serving
- Template web-db-user: **React 19 + Tailwind 4 + wouter** front end, **Node/Express + tRPC** back end, **Drizzle + managed MySQL** for persistence. Served as one container (Dockerfile, `/api/health`).
- New dependencies: `three`, `@react-three/fiber`, `@react-three/drei` (3D background + spacecraft viewer).
- No login required: players enter a callsign when submitting quiz scores, mission results or scenarios (keeps it frictionless; light server validation + rate limiting by length/score bounds).

## Data model (MySQL)
- `quiz_scores`: id, playerName, score, correct, total, levelReached, hintsUsed, durationSec, createdAt.
- `mission_runs`: id, teamName, players(json), scenarioKey, difficulty, design(json), score, outcome, breakdown(json), createdAt. Supports leaderboards, sharing (`/lab/run/:id`) and compare.
- `scenarios`: id, title, author, summary, difficulty, config(json), plays, createdAt.

## Game engine (pure TypeScript, `client/src/game/`)
- `catalog.ts`: components with simple/advanced tiers — structure, solar arrays, batteries, RTG, comms (UHF/S/X/Ka/optical), computers, ADCS, propulsion, thermal; instruments (camera, spectrometer, radar, magnetometer, IR telescope, lidar…) each with mass, power, cost, data rate, science value, objective affinity. Launch vehicles (Electron-, Falcon 9-, Falcon Heavy-, SLS-, Vega-class style generic names with capacities per orbit) and orbits (LEO, SSO, GEO, Molniya-type HEO, L2, lunar, Mars transfer) with explanations.
- `scenarios.ts`: built-in scenarios (Earth climate observer, Lunar south pole prospector, Mars atmosphere orbiter, Deep-space L2 telescope, CubeSat tech demo, Sandbox with no limits).
- `engine.ts`: computes budgets (mass incl. margin, power generation vs. load incl. eclipse/battery, data volume vs. downlink, cost, schedule), launch compatibility checks, and a simulated mission timeline with seeded random events (solar flare, reaction-wheel failure, launch delay, budget cut, ground-station outage, discovery bonus). Scores: mission success, science return, resource efficiency, decision quality, collaboration. Produces a report with "why" explanations.
- Team mode (**online multiplayer across devices**, approved by the user): 1–4 players in a shared room, roles Mission Planner / Spacecraft Engineer / Science Lead / Comms & Resources. Each design section is owned by a role; players who do not own a section negotiate by submitting a **proposal** that the team votes on (majority applies it). A Go/No-Go poll gates launch. Collaboration score reflects role coverage, balanced contributions, resolved proposals and consensus.

## Online multiplayer architecture
Socket.IO is attached to the same Express HTTP server at path `/api/socket.io` (so the gateway routes it as API; `destroyUpgrade:false` so Vite HMR keeps working). The server is authoritative: it holds room state in memory, validates role ownership on every edit, runs the shared mission engine for the simulation, saves the result to `mission_runs`, and broadcasts full room snapshots (`room:state`) after each change. Rooms are identified by a 5-character invite code / link `/lab/room/:code`; each player gets a seat token kept in localStorage so a refresh or dropped connection reclaims the same seat. Room snapshots are persisted to a `mission_rooms` table (debounced) so a server restart can rehydrate a room. Team chat is part of the room. The shared engine lives in `shared/game/` so the server recomputes every score (solo submissions are also re-simulated server-side from design + seed, preventing forged leaderboard entries). Quiz answers are likewise re-scored on the server from `shared/quiz.ts`.

## Pages / routes
`/` Home · `/nasa` Agency & insignia · `/missions` Missions timeline · `/academy` Quiz · `/leaderboard` Dashboard (quiz + missions, charts) · `/lab` Mission Lab hub · `/lab/play/:scenario` game · `/lab/run/:id` saved report · `/studio` Scenario Studio + community list · `/tutorials` · `/resources` (the 9 reference links).

## Project structure
- `client/src/components/space/` — SpaceBackground (R3F planets + scroll), Spacecraft3D viewer.
- `client/src/components/layout/` — NavBar, Footer, PageHeader, Logo.
- `client/src/content/` — NASA facts, missions, quiz bank (5 levels × 8 questions), tutorials.
- `client/src/game/` — catalog, scenarios, engine, types.
- `client/src/pages/` — page components above.
- `server/routers.ts` + `server/db.ts` — quiz, missions, scenarios procedures.

## Design
Candidate styles: (a) Swiss "Mission Control Brutalism" — monochrome grids, telemetry readouts, one signal color (p≈0.07); (b) Retro NASA 1970s Graphics Standards Manual — "worm" curves, flat red & white (p≈0.05); (c) Holographic glassmorphism HUD (p≈0.08). **Chosen: (a) blended with the 1975 NASA Graphics Standards Manual discipline.**

- **Design Movement**: International Typographic Style meets flight-control console (NASA Graphics Standards Manual, 1975).
- **Core Principles**: Precision over decoration; data is the ornament; generous negative space letting the 3D sky breathe; every number has a unit.
- **Color Philosophy**: Deep-space ink (#05070d) and lunar white carry the page; **Insignia Red (#FC3D21)** marks action and danger; telemetry cyan (#5ce1e6) for live data; amber for warnings. Feels serious, calm, mission-critical.
- **Layout Paradigm**: Asymmetric editorial layout: left-aligned oversized headlines, a thin vertical "mission rail" with section indices (01, 02…), content offset to a 7/12 column, sticky side readouts.
- **Signature Elements**: Coordinate tick marks / crosshair corners on panels; monospace telemetry labels (T+00:12, Δv); orbital-arc dividers.
- **Interaction Philosophy**: Instrument-like feedback — toggles click into place, values count up, warnings pulse only when constraints are violated.
- **Animation**: Planets drift continuously; scroll interpolates camera and planet set; sections fade/slide 24px on enter (400ms, ease-out); gauges animate value changes (300ms); no bouncy easing.
- **Typography System**: Display "Space Grotesk" (700, tight tracking, uppercase eyebrows); body "Inter"; data "JetBrains Mono". H1 64–96px, eyebrow 12px mono 0.2em tracking.
- **Brand Essence**: A mission-design academy for aspiring engineers that turns NASA-style trade-offs into a playable system. Precise, ambitious, credible.
- **Brand Voice**: Flight-director confident. "Every gram has a price. Spend it wisely." / "Go for launch — or find out why not."
- **Wordmark & Logo**: "BEYOND SIGNAL" in wide geometric caps (SIGNAL in Insignia Red) with an orbital ellipse slicing through the "O" and a red vector chevron (a nod to the NASA meatball's swoosh, without copying it).
- **Signature Brand Color**: Insignia Red #FC3D21.

NASA's insignia is shown on the agency page as historical/educational content with a "not affiliated with NASA" notice; mission imagery uses public-domain NASA photos from images.nasa.gov.
