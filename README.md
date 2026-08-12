# Server Directory

A live, self-contained Roblox server browser built on [Fluent](https://github.com/LostMorality/Fluent-modded). Instead of trusting Roblox's own (frequently useless) `ping` field, it ranks servers by real geographic distance to the nearest datacenter, powered by [RoValra's](https://github.com/NotValra/RoValra) public API — then merges in live player counts from Roblox itself.

![Server Directory preview](preview.png)

## Features

- **Real player counts** — pulled directly from `games.roblox.com/v1/games/{placeId}/servers/Public`, the same source Roblox's own server list uses.
- **Distance-based region ranking** — fetches all known Roblox datacenters and their coordinates, auto-geolocates you, and ranks every server by haversine distance to its datacenter. This is a far more honest latency proxy than Roblox's self-reported `ping` value, which is frequently `0` or wildly inaccurate.
- **Manual region override** — don't trust auto-geolocation, or just want to browse a specific region? Pick any datacenter city from the dropdown and instantly re-rank.
- **Sort & filter** — Fewest Players, Most Players, or Closest Region, with a toggle to hide full servers.
- **One-click Join / Share** — join a server directly, or copy a ready-to-run `TeleportToPlaceInstance` snippet to your clipboard.
- **Self-healing refresh loop** — exponential backoff on repeated fetch failures, incremental UI updates (no flicker/rebuild), and a watchdog that tears itself down cleanly if the window is closed.

## How it works

1. On open, the script fetches [`/v1/datacenters/list`](https://apis.rovalra.com/docs#/Datacenters/get_datacenters) from RoValra's API to get every known Roblox datacenter and its latitude/longitude.
2. It calls [`/v1/geolocation`](https://apis.rovalra.com/docs) (no IP argument needed — it resolves the requesting client automatically) to get your approximate location.
3. Every datacenter is ranked by [haversine distance](https://en.wikipedia.org/wiki/Haversine_formula) from you, closest first.
4. In the background, it queries [`/v1/servers/region`](https://apis.rovalra.com/docs#/Servers/get_servers_by_region) per city (closest first) to build a server-UUID → region cache. This cache is rate-limited and refreshed periodically, independent of the main player-count loop.
5. Meanwhile, the main loop polls Roblox's own public server list for live `playing` / `maxPlayers` counts.
6. The two datasets are merged by matching server UUID, giving you a list that's sorted by *actual* proximity while still showing *real* player counts — something neither API provides on its own.

## Installation

Server Directory is built as a plugin for [Infinite Yield](https://github.com/EdgeIY/infiniteyield). Save `ServerDirectory.iy` alongside your other IY plugins, or load it directly:

```lua
loadstring(game:HttpGet("https://raw.githubusercontent.com/<your-username>/Server-Directory/main/ServerDirectory.iy"))()
```

Once loaded, run:

```
;sd
```

or `;serverdirectory` to open the window.

## Requirements

- An executor with `request` / `http_request` / `syn.request` support (falls back to `game:HttpGet` if none are available)
- [Fluent (modded)](https://github.com/LostMorality/Fluent-modded) UI library (loaded automatically)
- [Infinite Yield](https://github.com/EdgeIY/infiniteyield) as the plugin host

## Credits

- [RoValra](https://github.com/NotValra/RoValra) / [apis.rovalra.com](https://apis.rovalra.com) — the public API providing datacenter locations, geolocation, and per-region server data that makes accurate distance-based ranking possible. This project would not exist without their work.
- [Fluent (modded)](https://github.com/LostMorality/Fluent-modded) — the UI framework used for the window.

## License

Licensed under the [GNU General Public License v3.0](LICENSE).
