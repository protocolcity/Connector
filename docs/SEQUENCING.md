# Connector — Sequencing Gates

**Date:** 2026-07-27 · **Reviewed:** 2026-09-12 (conn-38) · **Status:** living document
**Author:** reed · Connector Desk

---

## What this file is

A single reference so workers do not start network auth implementation ahead of
the gates that make it safe. **Design is always free.** Code that enforces auth,
mints tokens, or puts a port on the network is not free until the gates below
are cleared and You give explicit go.

Construction-order doctrine: **rooms → buildings → roads.** Connector
is the road. It coordinates and designs now; it implements when the rooms and
buildings are ready.

---

## Master gate checklist

Update this list in place as gates clear. Never uncheck a gate to re-open it —
file a new ticket for any regression.

| # | Gate | Status | Clears |
|---|---|---|---|
| **G1** | **WorkLane public source promotion** | ✅ done 2026-09-12: canonical public protocolcity/WorkLane, merged PR5. This establishes source promotion, not network authentication readiness. | Source-promotion dependency; other auth gates remain required. |
| **G2** | **Identity unhardcoding** — `WL_AGENT_ID` per-citizen config; neutral fallback `you`; PROCESS §5.2 citizen-hand rows land in WorkLane | ⬜ open | Everything that names a second human citizen. Hard blocker: without this, a second MCP connection inherits default host identity. |
| **G3** | **Multi-citizen design decisions connector needs** — CITIZENS.md schema, join flow, transport topology decided | ✅ done (design-complete, captured in JOIN_DESIGN.md) | Join config template draft; invite artifact design |
| **G4** | **You Q1 decision — auth tier for v1 multi-citizen** (JOIN_DESIGN.md §6 Q1): trusted LAN (no enforcement) vs. bearer tokens at proxy from day one | ⬜ open | Determines whether the join config template includes a `WL_TOKEN` field; determines scope of the WorkLane auth middleware ticket |
| **G5** | **You Q2 decision — connector packaging shape** (JOIN_DESIGN.md §6 Q2): static fill-in template in `docs/` vs. `wl connect <host>` command | ⬜ open | Determines the v1 join artifact shape; static template can be drafted before code; CLI command waits on identity unhardcoding |
| **G6** | **You Q3 decision — dashboard doors on shared host** (JOIN_DESIGN.md §6 Q3): shared host URLs vs. each citizen runs local dashboards against shared store | ⬜ open | Determines what "door URL" means in the invite artifact |
| **G7** | **Public brand ratified** — Connector brand confirmed, no rename tax | ✅ done | All copy and marketing may use "Connector" as the public name |
| **G8** | **Local multi-citizen dogfood** — auth enforcement proven on one machine with two citizen hands before any remote host is stood up | ⬜ open (depends on G1 + G2 + G4) | Remote-host provisioning; any WAN or untrusted-network auth work |
| **G9** | **PyPI name reserve `protocolcity-connector`** — only when shipping is imminent | ⬜ open (defer until ship pressure) | Package publish; not needed for design or local dogfood |

---

### Current identity evidence

Explicit local MCP authors and a dedicated implementation-worker identity have
been demonstrated. That does not prove authenticated multi-citizen isolation,
revocation, or remote transport. G2 remains open for those acceptance criteria;
the earlier implication that every MCP connection must inherit one identity is
obsolete. G4–G9 retain their individual decisions and evidence requirements.
No network service is enabled by this documentation update.

## Project-space chat — DITCHED (city-operator 2026-07-30)

Per-project agent chat / project space is **abandoned**.
BP coordination stack: **host chat** (main entry) + **WorkLane** (work orders) +
**Map** (ops glass). Multi-person later = **shared workspace folder**, not
in-suite rooms. Connector remains the **road** (host/join) when G1+ clear —
not a chat product.

**Code:** Phase 1 implementation (`room_server.py`, `hand.py`, `nostr_core.py`
+ tests) moved to `connector/archive/room-phase1/`.
`nostr_core.py` contains BIP-340 Schnorr / NIP-01 primitives worth reading if
the road layer ever needs signed identity — see `archive/room-phase1/SALVAGE.md`.

---

## Implementation chunks — what unlocks when

Each chunk names its gate set. Do not file an implementation ticket until all
named gates are cleared **and** You explicitly un-table it.

### Chunk 1 — Join config template (connector-local) ✅ DELIVERED

**Deliverable:** `docs/JOIN_CONFIG_TEMPLATE.md` — a static fill-in the Mayor
hands a new citizen.
**Gates:** G3 ✅ · G5 open (Q2 decision — 🔲 fields annotated in template; does not block use)
**Delivered:** 2026-08-11 · [`docs/JOIN_CONFIG_TEMPLATE.md`](JOIN_CONFIG_TEMPLATE.md) committed.
**File in:** `connector` store

### Chunk 2 — WorkLane auth middleware slice

**Deliverable:** WorkLane server reads `WL_TOKEN` header; 401 on mismatch;
localhost bypass rule; ticket in `worklane` store.
**Gates:** G1 · G2 · G4 (Q1 decision on auth tier)
**File in:** `worklane` store (one ticket per neighborhood)

### Chunk 3 — Token minting ceremony (citizen bearer tokens)

**Deliverable:** WorkLane issues citizen token at invite time; rotation on
revocation.
**Gates:** G1 · G2 · G4 · (Chunk 2 landed)
**File in:** `worklane` store

### Chunk 4 — Worker service tokens

**Deliverable:** service token minted at employment (PROCESS §5.2 row write);
stored in keychain; carried in `run.sh`.
**Gates:** G1 · G2 · (Chunk 2 landed)
**File in:** `worklane` store (token minting); `workforce` store (Roster hook)

### Chunk 5 — WorkForce auth check (:8797)

**Deliverable:** WorkForce Roster HTTP surface checks bearer tokens.
**Gates:** G1 · G2 · G4 · (Chunk 2 pattern established first)
**File in:** `workforce` store

### Chunk 6 — Invite artifact (Mayor-hands-citizen)

**Deliverable:** the full invite kit the Mayor hands a new citizen — MCP config
block + door URLs + token (if G4 chose enforcement).
**Gates:** G2 · G4 · G5 · G6
**File in:** `connector` store

### Chunk 7 — Remote host / WAN context

**Deliverable:** reverse-proxy reference config (nginx / Caddy) for WAN or
untrusted-network deployment; connector provides config snippet.
**Gates:** G1 · G2 · G4 · G8 (local dogfood complete first)
**File in:** `connector` store (config snippet); `ProtocolCity` (FOUNDING.md
"Inviting a citizen (networked)" section)

### Chunk 8 — `wl connect` CLI command

**Deliverable:** `wl connect <host>` generates the MCP config block and prints
door URLs.
**Gates:** G1 · G2 · G5 (Q2 decision chose CLI)
**File in:** `worklane` store (wl CLI is WorkLane's binary)

### Chunk 9 — First intake worker (Outlook proving ground)

**Deliverable:** `intake-outlook` worker employed; CONTRACT; ledger; dedupe on
thread-id.
**Gates:** G1 · G2 · INTAKE_WORKERS.md CONTRACT template confirmed stable ·
You green-light on external MCP dependency
**File in:** `connector` store for employment slip; `worklane` store for token
slice

---

## How to update this file

When a gate clears:
1. Change ⬜ open → ✅ done and note the date and ticket in the cell.
2. Reassess the chunks that depended on it — if all their gates are now ✅,
 note that the chunk is ready to file.
3. Do not file the implementation ticket from here — that is a You call per
 O-3; update the sequencing note in this doc and leave the ticket filing to
 the dispatch session.

When a new dependency surfaces:
- Add a row to the gate table (next G-number).
- Add the gate to any chunk it affects.
- Note the ticket or decision that introduced it.

---

## Sources

- `connector/docs/JOIN_DESIGN.md` — canonical join flow; Q1–Q3 open
 questions; implementation slices table
- `connector/docs/AUTH_ROAD.md` — §7 dependencies + sequencing;
 threat model; papers envelope; implementation follow-up tickets by engine
- `connector/docs/INTAKE_WORKERS.md` — §6 sequencing + gates for
 intake worker pattern
- `connector/AGENTS.md` — construction-order doctrine; product carve;
 sequencing (honest) section
- City-root `AGENTS.md` — connector row
- Identity unhardcoding (hard blocker, Gate G2)
- Multi-citizen design (design complete, Gate G3)
- Living gate list — close when first implementation epic is filed with You go
