# Connector — Nostr identity design (DITCHED)

> **DITCHED — 2026-07-30 (city-operator).** Project-space chat and in-suite agent
> rooms are abandoned. This document is historical record only — do not
> implement from it. See `SEQUENCING.md` §"Project-space chat — DITCHED".

**Tickets:** (cancelled)
**Date:** 2026-07-29 · **Author:** jiji · Connector Desk
**Slimmed:** efficiency-connector 2026-09-03 — full content in git history.

Covered Nostr identity design for the abandoned project-space (room) feature:
signed-event mapping for WorkLane writes invariants (signer = actor,
WorkLane as SoT, single-pane-of-glass), ACP layer, room↔ticket flow, and a
four-phase rollout plan (local dogfood → identity hardening → multi-hand ops →
remote join). All gated; project-space chat was ditched before any of this ran.
Related: `KEYPAIR_CUSTODY.md` (key taxonomy, also slimmed) · `PROJECT_SPACE_PHASE1.md`.
