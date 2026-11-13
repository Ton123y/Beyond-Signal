# Beyond Signal — TODO

## 1. Professional multipage NASA website
- [x] Multipage site (React front end, Node/Express back end): Home, NASA agency, Missions, Academy (quiz), Leaderboard dashboard, Mission Lab, Scenario Studio, Tutorials, Resources.
- [x] "More info about NASA and its logo": agency history eras, facts, mission directorates, field centers, science divisions, and the NASA insignia (meatball, worm, seal) with history and symbolism.
- [x] Missions timeline with NASA photos, categories and filters.
- [x] Professional, standout design (not for kids); distinctive Beyond Signal wordmark; "not affiliated with NASA" notice.

## 2. Dynamic space background
- [x] Background of the web is planets moving; the more you scroll, the more the planets/sky change (scroll-driven 3D scene).

## 3. NASA quiz game (Mission Academy)
- [x] Simple questions about NASA across 5 levels.
- [x] Every answered question gives points (harder levels worth more; streak bonus).
- [x] Hints about the answers (using a hint reduces points) and explanations after answering.
- [x] Players submit their answers with a callsign; score computed on the server.

## 4. Dashboard / leaderboard
- [x] Dashboard of people who submit their answers: quiz leaderboard, stats and charts.
- [x] Mission leaderboards / rankings by scenario.

## 5. 3D Space Mission Design Game (Mission Lab)
- [x] Define a mission objective and requirements before engineering decisions (scenario briefings).
- [x] Design a spacecraft: structure, power systems, batteries, communications, onboard computer, attitude control, propulsion, thermal, scientific instruments; simple vs advanced components depending on difficulty.
- [x] Instruments with mass, power, cost, data generation and science value.
- [x] Choose launch vehicle (payload capacity, cost, orbit capability) and orbit (explained: coverage, observation, communications, lifetime); check spacecraft mass and orbit are compatible with the launch vehicle.
- [x] Clearly display resources in a real-time dashboard: mass, power, budget, data, communications, time (development + lifetime).
- [x] Every major decision has a consequence; trade-offs; no single perfect solution.
- [x] Interactive 3D spacecraft visualization reflecting the design; 3D orbit/launch simulation view.
- [x] Mission simulation with random events / unexpected constraints; difficulty levels; multiple scenarios; sandbox mode.
- [x] Scoring rewards mission success, scientific value, efficient resource use, decision making and collaboration.
- [x] Mission report at end with post-mission analysis explaining why it succeeded or failed; modify the design and simulate multiple times.
- [x] Save, share and compare mission designs; leaderboards.
- [x] Simple-language explanations of concepts (orbital altitude, payload capacity, power, data rate, comms) inside the game.

## 6. Online multiplayer (up to 4 players across devices)
- [x] Single-player and online team play: create/join a room by code or link from different devices, up to 4 participants on the same mission.
- [x] Roles (Mission Planner, Spacecraft Engineer, Science Lead, Comms & Resources) owning different decisions; real-time shared design.
- [x] Negotiation between competing priorities: proposals + team votes; team chat; Go/No-Go poll before launch.
- [x] Final result depends on team coordination (collaboration score); reconnect keeps the seat.

## 7. Scenario Studio (make and design your own space mission games)
- [x] People can create and design their own mission scenarios (objective, budget, constraints, destinations, required instruments, events, difficulty), save them, and others can play them solo or online.

## 8. Tutorials and resources
- [x] Tutorials to help people play the game and make their own mission games.
- [x] Resources page using the provided reference links (NASA mission design PDF, NASA software GSC-19468-1, JPL dataverse, GSFC Mission Design Lab, mission-viz, Webb build-it-yourself, CubeSat 101, Space Apps VR challenge, CSA games).

## 9. Delivery
- [x] Database tables migrated; health route; route manifest; logo metadata; checkpoint saved.
- [x] Validation pass by a read-only review agent.

## Evidence
- `npx tsc --noEmit` clean; production `vite build` succeeds.
- Headless runs: all 13 routes 200; Home/Lab/Play/Academy/Studio/Agency load with 0 console errors.
- End-to-end (scripts/smoke_flow.py): solo design → review → launch → 3D playback → report; two separate browser contexts create/join a room by code, claim roles, live-edit, propose → approve, GO/No-Go, launch, both see the shared report. Test rows removed afterwards.
- Review-agent findings fixed: server-derived solo collaboration (signed iteration token + design review findings), ranked-only dashboard stats, spoof-resistant rate-limit keys + Socket.IO event quotas, multi-select Compare picker.
