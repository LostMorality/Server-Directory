# Server Directory

A live, self-contained Roblox server browser built on [Fluent](https://github.com/LostMorality/Fluent-modded). Instead of trusting Roblox's own (frequently useless) `ping` field, it ranks servers by real geographic distance from you to each datacenter, powered by [RoValra's](https://github.com/NotValra/RoValra) public API — then merges in live player counts from Roblox itself, and displays an estimated ping calibrated against your actual measured latency.

![Server Directory preview](preview.png)

## Features

- **Real player counts** — pulled directly from `games.roblox.com/v1/games/{placeId}/servers/Public`, the same source Roblox's own server list uses.
- **Distance-based region ranking** — geolocates you (not the server you happen to be on — Roblox doesn't guarantee proximity matchmaking), fetches all known Roblox datacenters and their coordinates, and ranks every server by haversine distance from you. This is a far more honest latency proxy than Roblox's self-reported `ping` value, which is frequently `0` or wildly inaccurate.
- **Self-calibrating ping estimate** — reads your actual measured ping to your current server (via Roblox's own `Stats` service) and uses it to calibrate the distance→ping model, instead of trusting a blind theoretical constant.
- **Sort & filter** — Fewest Players, Most Players, or Closest Region, with a toggle to hide full servers.
- **One-click Join / Share** — join a server directly, or copy a ready-to-run `TeleportToPlaceInstance` snippet to your clipboard.
- **Self-healing refresh loop** — exponential backoff on repeated fetch failures, incremental UI updates (no flicker/rebuild), and a watchdog that tears itself down cleanly if the window is closed.

## How it works

1. On open, the script fetches [`/v1/datacenters/list`](https://apis.rovalra.com/docs#/Datacenters/get_datacenters) from RoValra's API to get every known Roblox datacenter and its latitude/longitude.
2. It geolocates *you* via [ipapi.co](https://ipapi.co) (RoValra's own `/v1/geolocation` endpoint only resolves known Roblox server IPs, not arbitrary player IPs, so it can't be used for this — this is the working substitute). If that fails, it falls back to using your current server's own datacenter (via RoValra's [`/v1/servers/details`](https://apis.rovalra.com/docs), keyed by `game.JobId`) as a rough proxy.
3. Every datacenter is ranked by [haversine distance](https://en.wikipedia.org/wiki/Haversine_formula) from that origin, closest first.
4. To turn distance into an estimated ping, it reads your **actual measured ping** to your current server from Roblox's own `Stats.Network.ServerStatsItem["Data Ping"]` and your distance to that server's datacenter, then solves for a personalized coefficient in `ping = coefficient * sqrt(distance) + baseOverhead` (square-root, not linear — real-world latency scales sub-linearly with distance since long-haul backbone/submarine routes are far more efficient per km than short local hops; a linear fit calibrated from a single short hop badly overshoots on long ones). That coefficient is then applied to every other datacenter's distance. Your current server's row shows the real measured value directly, not an estimate.
5. In the background, it queries [`/v1/servers/region`](https://apis.rovalra.com/docs#/Servers/get_servers_by_region) per city (closest first) to build a server-UUID → region cache. This cache is rate-limited and refreshed periodically, independent of the main player-count loop.
6. Meanwhile, the main loop polls Roblox's own public server list for live `playing` / `maxPlayers` counts.
7. The datasets are merged by matching server UUID, giving you a list that's sorted by *actual* proximity while still showing *real* player counts and a calibrated ping estimate — something none of these APIs provide on their own.

## Installation

Server Directory is a plugin for [Infinite Yield](https://github.com/EdgeIY/infiniteyield) — it isn't loaded with its own `loadstring`, it's added through IY's own plugin system:

1. [Download `ServerDirectory.iy`](ServerDirectory.iy) from this repo.
2. Move the file into your executor's `workspace` folder (the same folder IY itself reads plugins from).
3. In-game, with Infinite Yield running, open **Settings → Plugins → Add Plugin**, type `ServerDirectory.iy`, and click **Add**.
4. Run `;sd` or `;serverdirectory` to open the window.

## Requirements

- An executor with `request` / `http_request` / `syn.request` support (falls back to `game:HttpGet` if none are available)
- [Fluent (modded)](https://github.com/LostMorality/Fluent-modded) UI library (loaded automatically)
- [Infinite Yield](https://github.com/EdgeIY/infiniteyield) as the plugin host

## Credits

- [RoValra](https://github.com/NotValra/RoValra) / [apis.rovalra.com](https://apis.rovalra.com) — the public API providing datacenter locations, geolocation, and per-region server data that makes accurate distance-based ranking possible. This project would not exist without their work.
- [Fluent (modded)](https://github.com/LostMorality/Fluent-modded) — the UI framework used for the window.

## License

Licensed under the [GNU General Public License v3.0](LICENSE).
