# Connector — Keypair Custody Design

> **DITCHED — 2026-07-30 (city-operator).** Project-space chat and in-suite agent
> rooms are abandoned. This keypair custody design (Nostr keys, delegation format,
> worker npub issuance) was written for that architecture — it is historical record
> only. Do not implement from it. See `SEQUENCING.md` §"Project-space chat — DITCHED".

**Date:** 2026-07-29 · **Status:** DITCHED (city-operator 2026-07-30)
Historical record: Keypair issuance and custody design for the abandoned Nostr
room architecture. Covered four key classes (You, Hand, Worker,
Service), storage tiers (client-side file → keychain → NIP-07 extension), worker
keypair provisioning at hire time, and NIP-26-based delegation events scoped per
project. DITCHED before any implementation. Full design accessible via git history.
