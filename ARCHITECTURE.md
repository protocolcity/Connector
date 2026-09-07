# Connector — Architecture

**Status:** binding L1-adjacent paper (workspace wave · city-operator 2026-08-04)
**Last verified:** 2026-08-31 · **jiji** slimmed `AGENTS.md` to L1 CORE + pointers; layers / SoT / invariants unchanged; SEQUENCING gates unchanged
**Public brand:** Connector · **Store / prefix:** `connector` / `conn-`
**Package intent (not shipped):** `protocolcity-connector`

This paper is the structure agents build against. Design notes under `docs/`
explain *why* and *when*; this file states *what is*, *who owns which fact*,
and *what must not be violated*. Structural changes update this paper in the
same close-out (see §4).

---

## 1. Layers — what talks to what

Connector today is a **design + law + bootstrap neighborhood**, not a running
product binary. Solo-laptop city v1 does **not** require Connector to ship.
Construction-order doctrine: **rooms → buildings → roads** — this
product is the road layer.

```
┌─ Citizen / hands (host chat · MCP / wl · WorkForce scheduled seats) ─┐
│  Claim / design / coordinate under project=connector                 │
└──────────────────────────────┬───────────────────────────────────────┘
                               │ WorkLane Desk HTTP (:8799)
                               ▼
                    ┌──────────────────────┐
                    │ connector store      │  ← system of record for WOs
                    │ (prefix conn-)       │
                    └──────────────────────┘
                               │
          ┌────────────────────┼────────────────────┐
          ▼                    ▼                    ▼
   ┌─────────────┐    ┌─────────────────┐   ┌──────────────────┐
   │ L1 law      │    │ Design vault    │   │ Hands / jobs     │
   │ AGENTS.md   │    │ docs/*          │   │ workers/*        │
   │ this paper  │    │ (not runtime)   │   │ jiji · jobs      │
   └─────────────┘    └─────────────────┘   └──────────────────┘
          │
          │  (not product surfaces)
          ▼
   ┌─────────────────┐   ┌──────────────────────────────┐
   │ prototypes/     │   │ archive/room-phase1/         │
   │ nostr-room      │   │ salvaged crypto reference    │
   │ RETIRED PoC     │   │ NOT active code              │
   └─────────────────┘   └──────────────────────────────┘
```

### Layer inventory (as on disk)

| Layer | Path / surface | Role today |
|---|---|---|
| **L1 law** | `AGENTS.md`, this `ARCHITECTURE.md` | Binding product carve, brand, non-negotiables, structure |
| **L1 programs** | `PROGRAMS.md` | Named lines of work (programs layer); gate/state summary per program |
| **Vendor entry** | `CLAUDE.md`, `GROK.md` | Point at `AGENTS.md` only |
| **Design vault** | `docs/` | Join, auth-road interface, sequencing gates, intake-worker doctrine; several **DITCHED** historical papers (project-space era) |
| **Hands** | `workers/jiji/` | Active claiming lane (`worker:jiji`) — design/law/bootstrap |
| **Jobs** | `workers/efficiency-connector/` | Scheduled efficiency pass (kind=`job`, does not drain backlog seats) |
| **Retired seat papers** | `workers/reed/`, `workers/zach/` | Historical CONTRACT/prompt only — not hired seats |
| **Desk join** | `.protocolcity/desk-join.json` | Product slug/prefix/display + desk URL for city registry |
| **Work orders** | WorkLane store `connector` (`conn-*`) | Ticket lifecycle; PROCESS.md owns the protocol |
| **Prototypes** | `prototypes/nostr-room/` | **Retired** Nostr room PoC — do not run |
| **Archive** | `archive/room-phase1/` | Ditched project-space Phase 1; salvage notes for `nostr_core` primitives only |
| **Local scratch** | `local/` | Gitignored/runtime reports; not product SoT |

### What is deliberately *not* here

| Absent | Why |
|---|---|
| Production server / package source tree | Package name reserved as intent only; ship gated (SEQUENCING G9) |
| Auth enforcement code | Lives on engine ports (WorkLane, WorkForce, suite) when un-tabled — never in connector guts |
| Join CLI / `wl connect` | Future; `wl` is WorkLane's binary (Chunk 8 in `docs/SEQUENCING.md`) |
| Live credentials / tokens | Forbidden in git (AGENTS non-negotiable) |
| In-suite project-space chat | Ditched 2026-07-30 — host chat + WorkLane + Map |

### External seams (connector coordinates; others own)

| Seam | Owner | Connector's job |
|---|---|---|
| Ticket store + MCP / `wl_*` | **WorkLane** | Consume with `project=connector`; design invite / identity packaging |
| Roster / employment | **WorkForce** | Seat papers under `workers/`; never edit `workforce/local/roster.json` from this seat |
| Port auth checks | **Each engine** | Define paper *format* in design docs only until gates clear |
| Suite Map / Desk UI | **ProtocolCity / suite** | Not a fifth room; no Map forms for create/claim/close |
| City product registry | **City root AGENTS / desk-join** | `.protocolcity/desk-join.json` is the local join receipt |

---

## 2. Single source of truth — one owner per fact

| Domain of state | Single owner | Not authoritative |
|---|---|---|
| Product carve, brand string **Connector**, non-negotiables | `AGENTS.md` | Marketing copy elsewhere; old persona names in archive |
| Layers, SoT map, invariants, change rule | **This file** (`ARCHITECTURE.md`) | Chat summaries; re-derived agent plans |
| Work-order status, labels, ownership, comments | WorkLane **connector** store | Host chat; prototype room DBs; Map glass (viewer) |
| Sequencing gates (what may be implemented when) | `docs/SEQUENCING.md` | Individual design docs' aspirational "next" sections |
| Join UX design (citizen steps, open You Qs) | `docs/JOIN_DESIGN.md` | Prototype READMEs |
| Auth *interface* (papers envelope; engines check) | `docs/AUTH_ROAD.md` | Connector inventing engine middleware |
| Intake-worker doctrine (employee under contract) | `docs/INTAKE_WORKERS.md` | Zapier-style pipe metaphors |
| Hand employment terms / claimable feed | `workers/<id>/CONTRACT.md` | Roster UI alone; demo stubs |
| Desk product identity (slug, prefix, display) | `.protocolcity/desk-join.json` + city registry | Guessed prefixes in tickets |
| Ticket lifecycle rules | `worklane/PROCESS.md` | Ad-hoc close comments without §5 sections |
| Ditched project-space / Nostr room plans | `docs/SEQUENCING.md` (ditch notice) + `archive/` / ditched banners on old docs | Implementing from `NOSTR_*`, `ROOM_*`, `KEYPAIR_*`, `PROJECT_SPACE_*` as if live |
| Salvageable crypto primitives (reference only) | `archive/room-phase1/SALVAGE.md` + `nostr_core.py` there | `prototypes/nostr-room/` as product path |
| Live secrets / tokens / host credentials | **Outside git** (keychain / env / Mayor invite) | Any file under this tree |

---

## 3. Boundaries and invariants

Agents **must not** violate:

1. **Brand.** Public product name is **Connector** only. Road/transit
 language may appear in prose; never invent a second product name.

2. **Not a fifth suite room.** Not in-suite per-project agent chat. Coordination
 stack = host chat + WorkLane + Map. Multi-person later = shared workspace
 folder, not Connector chat servers.

3. **Not a pipe.** Future integrations are **employees under contract**
 (Track 2), not anonymous SaaS connectors. See `docs/INTAKE_WORKERS.md`.

4. **Port auth stays on engines.** Connector designs join packaging and paper
 format; WorkLane / WorkForce / suite enforce. Do not land production auth
 middleware, token minting, or open network ports from this neighborhood
 until `docs/SEQUENCING.md` gates clear **and** You un-table the chunk.

5. **One ticket per neighborhood.** Cross-engine work → sibling tickets in
 owning stores (`worklane`, `workforce`, …). Connector tickets stay join UX /
 design / law / bootstrap for *this* tree.

6. **Feed discipline.** Claiming lane drains **only** `worker:jiji` on store
 `connector`. Do not claim other products' tickets or demo-worker stubs.
 Efficiency job is not a backlog claimer.

7. **No secrets in git.** Credentials, bearer tokens, live host secrets never
 land in this repository.

8. **Prototypes and archive are not product.** Do not run `prototypes/nostr-room`
 or restore `archive/room-phase1` as live surfaces without a new city-operator-scoped
 ticket that rewrites this paper.

9. **Design is free; network auth implementation is not.** File and draft
 freely. Implementation chunks list gates in `docs/SEQUENCING.md` — respect
 them.

10. **For You scarcity.** Do not bare-`gate_type=human` to park work.
 Parked work uses deferred/parked notes per PROCESS.

11. **Public face.** Papers on disk + Desk store are binding. The public git
    face is **protocolcity/Connector**; push follows neighborhood practice.

---

## 4. Change rule

| Change type | Required update |
|---|---|
| New top-level layer, package, or runtime entrypoint | Update **§1** in this file same close-out |
| New domain of state or owner flip | Update **§2** same close-out |
| New hard boundary or retired surface | Update **§3** same close-out |
| Gate flip on sequencing | Update `docs/SEQUENCING.md` (living checklist); touch this paper only if layers/SoT/invariants change |
| Design-only doc under `docs/` | No ARCHITECTURE change unless it alters layers, SoT, or invariants |
| WorkForce hire / retire of a connector seat | CONTRACT under `workers/`; §1 hands row if active seats change |

**Rule:** structural change without an ARCHITECTURE.md update in the same
ticket close-out is incomplete work — leave the ticket open or file a child.

**Verify habit:** relative links in this paper resolve; content matches the tree
*as it is*, not a future roadmap.

---

## Canonical entrypoints (agents)

| Need | Open |
|---|---|
| Product law | [`AGENTS.md`](AGENTS.md) |
| This structure | [`ARCHITECTURE.md`](ARCHITECTURE.md) (this file) |
| When code is allowed | [`docs/SEQUENCING.md`](docs/SEQUENCING.md) |
| Join design | [`docs/JOIN_DESIGN.md`](docs/JOIN_DESIGN.md) |
| Auth seam design | [`docs/AUTH_ROAD.md`](docs/AUTH_ROAD.md) |
| Intake doctrine | [`docs/INTAKE_WORKERS.md`](docs/INTAKE_WORKERS.md) |
| Active hand contract | [`workers/jiji/CONTRACT.md`](workers/jiji/CONTRACT.md) |
| Ticket process | suite WorkLane PROCESS |

---

## Related city papers (link, do not duplicate)

- Product carve + rooms → buildings → roads
- Architecture-first plant / template wave
- `ProtocolCity/docs/research/multi-citizen-design-2026-07.md`
- `ProtocolCity/docs/research/beyond-solo-machine-2026-07.md` Track 2
- WorkLane identity lineage — gate for multi-citizen auth
