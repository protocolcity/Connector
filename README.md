# Connector

> Design and law neighborhood for **Protocol City** join, auth packaging, and
> sequencing. The installable package `protocolcity-connector` is **not shipped
> yet** — this repo is the public design face.

**Public brand:** Connector  
**Org repo:** [protocolcity/Connector](https://github.com/protocolcity/Connector)  
**Related suite:** [BluePrint](https://github.com/protocolcity/BluePrint) ·
[WorkLane](https://github.com/protocolcity/WorkLane) ·
[WorkForce](https://github.com/protocolcity/WorkForce)

## What this is

Connector is the **road layer**: how a citizen (You) joins a city folder on the
network, how invite/config packaging is designed, and when network auth may be
implemented. Construction-order doctrine: **rooms → buildings → roads.**
Solo-laptop city v1 does not require Connector to ship.

Port auth stays on each engine (WorkLane / WorkForce / suite). Connector owns
join UX design and paper format — not engine middleware.

## Start here

| Doc | Role |
|---|---|
| [`AGENTS.md`](AGENTS.md) | Product law / non-negotiables |
| [`ARCHITECTURE.md`](ARCHITECTURE.md) | Layers, SoT, invariants |
| [`PROGRAMS.md`](PROGRAMS.md) | Named programs + gate summary |
| [`docs/SEQUENCING.md`](docs/SEQUENCING.md) | When implementation is allowed |
| [`docs/JOIN_DESIGN.md`](docs/JOIN_DESIGN.md) | Join UX design |
| [`docs/AUTH_ROAD.md`](docs/AUTH_ROAD.md) | Auth seam (engines enforce) |
| [`docs/JOIN_CONFIG_TEMPLATE.md`](docs/JOIN_CONFIG_TEMPLATE.md) | Fill-in invite config |
| [`docs/INTAKE_WORKERS.md`](docs/INTAKE_WORKERS.md) | Integrations as employees |

Several papers under `docs/` are marked **DITCHED** (project-space / Nostr room
era) and kept as history only.

## Status

Design vault + law. No production Connector binary in this tree. Package publish
waits on sequencing gates and ship pressure.
