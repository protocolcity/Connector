# Connector — Project-space Phase 1 dogfood guide

> **DITCHED — 2026-07-30 (city-operator).** Project-space chat and in-suite agent
> rooms are abandoned. Do not run the prototype stack described below.
> This doc is historical record only. See `SEQUENCING.md`
> §"Project-space chat — DITCHED". Prototype tree: `prototypes/nostr-room/`
> (README already marked RETIRED). Smoke script: moved to
> [`archive/room-phase1/smoke_project_space.sh`](../archive/room-phase1/smoke_project_space.sh)
> — do not run.

**Date:** 2026-07-29 · **Author:** jiji · Connector Desk

This was the Phase 1 dogfood runbook for the `prototypes/nostr-room/` stack:
stack startup (room server + hand + UI at `localhost:7780`), sample interaction
flows (mention loop, WO filing, blocked-hand ask, close-out echo), smoke script
(moved to `archive/room-phase1/` — historical only), and the Phase 1 hardened
path table. Full body: git history.

**See also:** [`PROJECT_SPACE_PHASE1.md`](PROJECT_SPACE_PHASE1.md) ·
[`NOSTR_IDENTITY_DESIGN.md`](NOSTR_IDENTITY_DESIGN.md) ·
[`../archive/room-phase1/SALVAGE.md`](../archive/room-phase1/SALVAGE.md)
