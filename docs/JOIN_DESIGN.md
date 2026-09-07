# Connector — Join UX + Client Connect Design

**Date:** 2026-07-27 · **Status:** design draft; open
questions You-gated
**Author:** reed · Connector Desk

---

## What this doc is

Product carve: Connector owns join UX and client connect — the road a
citizen walks to reach a city on the network. This doc designs that road for v1.
It is a **design record**, not an implementation ticket. No production auth code
is written here; implementation slices follow per neighborhood when gates open.

**Construction-order doctrine:** rooms → buildings → roads. WorkLane flip and
identity unhardcoding are the gates. Connector coordinates and designs
now; it does not implement the network-auth layer ahead of those gates.

**Substrate this doc builds on:**

- Covenant — three tiers: *citizen* / *hand* / *worker*
- Triad — *You* / *citizen* / *city-operator* vocabulary
- Multi-citizen design — citizens registry, identity, trays, transport
- Beyond-solo-machine research — Track 1 shared server; Track 2
 connector workers as employees
- Identity unhardcoding — the single hard blocker to a second human

---

## 1. Invite / point-at-city flow

A new citizen joins a running city in four steps. Connector owns the UX that
packages and communicates these steps; engines own the enforcement surfaces
those steps depend on.

```
Step 1 GET THE FOLDER
 Clone the city repos, or join a shared checkout on the same machine.
 (git is the audit substrate; a citizen does not need to know that word
 to start — the joining copy they see says "get the folder.")

Step 2 SET YOUR IDENTITY
 Configure the WorkLane MCP with a personal identity string:
 author=<name>-<surface>
 Example: author=alex-desktop or author=alex-phone-terminal
 This string is the citizen's "hand name" — stable, human-readable,
 matches their CITIZENS.md entry. Not an email, not a UUID.

Step 3 CONNECT TO THE CITY STORE
 Point the MCP at the shared WorkLane server (HTTP):
 WL_MCP_URL=http://<host>:<port>
 The dashboards (Office / Desk / Roster) either run on the shared host
 or each citizen runs local dashboard instances against the shared URL.
 The store URL is the single source of truth either way.

Step 4 APPEAR ON THE ROSTER
 Once MCP is connected and identity is set, the citizen appears on the
 Roster as a citizen entry — distinct from worker rows.
 The Mayor adds them to CITIZENS.md (Mayor-gated act per §1).
```

**Connector's role in this flow:** packaging and communication. Connector does
not reimplement the store, the MCP server, or the Roster rendering. It owns:

- The copy / onboarding text the citizen reads ("how do I join?")
- The client config format the citizen drops into their terminal
- The invite artifact the Mayor hands to a new citizen

---

## 2. Identity model

### Tiers (Covenant)

| Tier | Who | Signs as | Clears gates? | Authority ceiling |
|---|---|---|---|---|
| **Citizen (You)** | A human of the city | Their hand name | Yes | Mayor / Owner / Citizen per hat |
| **Hand** | A terminal / app carrying You | `<name>-<surface>` | Yes (as You) | Inherits from citizen |
| **Worker** | An employed agent | `worker-id` (e.g. `jiji`) | No — files into tray | L2 CONTRACT |

A session runs under citizen authority (hand) or worker authority (worker-id) —
never both. The test is *whose authority the session runs under*, not what model
runs in it.

### Citizens registry (`CITIZENS.md`)

City-root paper, peer to `AGENTS.md`. One section per human citizen. Schema
(from §1):

```markdown
## <Display Name>

- **Role:** Mayor | Owner(<neighborhood>) | Citizen
- **Git identity:** Name <email>
- **Hands:**
 - `<name>-<surface>` — <description: terminal / app / host>
 - `<name>-<surface2>` — …
- **Since:** <date>
```

**Solo city default:** one citizen registered — the founding citizen — role
defaulting to Mayor. The `found` command should write this entry at
founding time.

**Gate:** `CITIZENS.md` additions are Mayor-gated. Workers may read; only a
citizen (acting as Mayor) may write. An invite is a city-law act.

### Identity unhardcoding dependency

The single hard implementation blocker: `default-host-identity` is today hardcoded
as the WorkLane default identity. Until WorkLane ships a per-citizen slot
(`WL_AGENT_ID` / `author` in MCP config, neutral fallback `you`), a second
human connecting the MCP inherits the city-operator identity. All multi-citizen
mechanics wait on this ticket.

**Connector does not implement identity unhardcoding** — that lives in the WorkLane
neighborhood. Connector's join flow is designed for the world after it
lands.

---

## 3. Hands — MCP config + wl CLI

Connector packages the **client configuration** a citizen or worker drops into
their terminal to connect their hands to the city. It does not reimplement the
store or the MCP server.

### WorkLane MCP config block (per hand)

```json
{
 "mcpServers": {
 "worklane": {
 "command": "wl",
 "args": ["mcp"],
 "env": {
 "WL_AGENT_ID": "<name>-<surface>",
 "WL_MCP_URL": "http://<host>:<port>"
 }
 }
 }
}
```

`WL_AGENT_ID` is the hand's identity string, matching the `CITIZENS.md` entry.
`WL_MCP_URL` is the shared WorkLane server URL (omit or set to
`http://localhost:<port>` for a solo-machine city).

### wl CLI

`wl` is the canonical WorkLane verb. Citizens use `wl` as they would in any local session; the only
difference is the `WL_MCP_URL` env var points at the shared host instead of
localhost.

```
# Ticket operations work identically once MCP is connected:
wl create "fix login flow" --project myneighborhood
wl claim 42 --project myneighborhood
wl close 42 ...
```

The MCP connection is transparent to the citizen's workflow.

### Connector packaging (design target, not implemented)

Connector's v1 deliverable is a **join config generator** — a short command or
artifact that:

1. Asks for the city host URL and the citizen's hand name
2. Outputs the MCP config block above, ready to paste into the citizen's
 `~/.claude/settings.json` (or equivalent)
3. Prints the door URLs (§4) the citizen can bookmark

Implementation of this generator is gated on identity unhardcoding and the broader network
auth decision (see §6 open questions).

---

## 4. Doors — room URLs after connect

Once a citizen connects, the three city rooms are reachable at these URLs.
For a shared host: substitute `localhost` with `<host>`.

| Room | URL | Function word | Powered by |
|---|---|---|---|
| **Office** | `http://<host>:8796` | Projects | ProtocolCity |
| **Desk** | `http://<host>:8799` | Tickets | WorkLane |
| **Roster** | `http://<host>:8797` | Workers | WorkForce |

Dashboard naming law (sixth amendment, 2026-07-15): rendered headers read
`[Folder] Office`, `[Folder] Desk`, `[Folder] Roster` — suite brand as
subtitle. Browser `<title>` tags carry full
`ProtocolCity — Desk · Tickets` form for tab identification.

**Connector's role:** surface the door URLs in the join artifact. Citizens
bookmark these; they are not part of the MCP config.

**Door auth surface:** the rooms today have no per-citizen auth. In a shared
LAN context, network access is the gate. For WAN or untrusted networks, a
reverse-proxy auth layer in front of the dashboards is the next step — not
designed here, flagged as an open question (§6).

---

## 5. Carve boundary — engines enforce; connector carries papers

This is the most important constraint:

> *"Port auth enforcement stays on each engine (WorkLane, WorkForce, BluePrint
> suite) — connector does not own store or runner guts."*

**What connector owns:**

- The joining story: copy, flow, invite artifact, client config
- The identity paper (hand name, CITIZENS.md entry template)
- The config a citizen carries when they connect (`WL_AGENT_ID`, `WL_MCP_URL`)
- The door list (room URLs) surfaced to the citizen after connect

**What connector does NOT own:**

- MCP server implementation (WorkLane)
- Port auth, bearer tokens, or TLS (each engine's surface)
- Roster rendering (WorkForce / Dispatch)
- Worker employment records (WorkForce roster)
- Dashboard room rendering (WorkForce / WorkLane)
- Ticket store guts (WorkLane)

**The metaphor:** Connector is the road and the papers the vehicle carries.
It is not the vehicle, not the checkpoint, not the building at the destination.
Road language (join, connect, invite) lives in Connector copy; it is not a
second product name.

**When work crosses the boundary:** file a sibling ticket in the owning store.
Connector ticket for join UX changes; WorkLane ticket for server-side auth
changes; WorkForce ticket for Roster rendering changes. One ticket per
neighborhood.

---

## 6. Open questions (You-gated)

These require a You decision before implementation tickets are runnable.
They are not blockers to this design doc; they are blockers to the first
implementation slice.

### Q1 — Auth tier for v1 multi-citizen

**Options:**

- **Trusted LAN (no auth):** any machine on the LAN connects; identity is
 conventional, not enforced. Acceptable for a trusted household or small
 team on a private network. Fastest path to a working multi-citizen city.
- **Bearer tokens at proxy:** per-citizen token minted at invite time; reverse
 proxy enforces before traffic reaches the WorkLane server. Enforces the
 gate; adds an invite-ceremony step.

**Recommendation (from /):** trusted LAN is acceptable v1 for a
known-team context. Bearer tokens are the next step when the city goes WAN or
the team grows beyond a trusted network.

**Decision needed from:** You. Answer determines whether the join config
generator includes a token field, and whether WorkLane's auth slice carries an auth surface or a separate ticket owns it.

### Q2 — Connector packaging shape

**Options:**

- A CLI command (`wl connect <host>`) that prints the config block and doors
- A static markdown template in `docs/` that the citizen fills in manually
- A short shell script bundled with Connector

**Recommendation:** start with a filled-in markdown template (no code to ship,
no gate dependency); graduate to a `wl connect` command after identity unhardcoding lands and
the auth story is clear.

**Decision needed from:** You. This is a product-shape call.

### Q3 — Dashboard doors on the shared host

Do the dashboards (Office / Desk / Roster) run on the shared host, or does
each citizen run local dashboard instances against the shared store URL?

**Recommendation (from §6):** both are valid; the store URL is the
single source of truth. Start with local instances (lower ops surface); migrate
to shared hosting when a citizen doesn't want to run their own dashboards.

**Decision needed from:** You. Determines what "door URL" means in the
invite artifact (shared host URL vs. "run local dashboard, point at shared
store").

---

## Implementation slices (design outputs — not filed from here)

These follow after the open questions above are resolved and identity unhardcoding lands.
Filing / routing is a You call per O-3.

| Neighborhood | Scope | Depends on |
|---|---|---|
| **WorkLane** | Identity parameterization (`WL_AGENT_ID`, neutral fallback `you`, citizen-hand rows in PROCESS §5.2) | First to land — everything else blocks on it |
| **connector** | Join config template (docs/JOIN_CONFIG_TEMPLATE.md) — a static fill-in citizen takes at invite time — **✅ delivered** 2026-08-11 | identity design (can draft before it ships) |
| **connector** | Invite artifact: what the Mayor hands a new citizen | Q1 auth decision + identity unhardcoding |
| **ProtocolCity** | CITIZENS.md template — add to templates/ | This design doc |
| **ProtocolCity** | FOUNDING.md: "Inviting a citizen" section | This design doc |
| **WorkForce** | Roster: Citizens panel distinct from worker rows | |

---

## Sources

- `ProtocolCity/docs/research/multi-citizen-design-2026-07.md`
- `ProtocolCity/docs/research/beyond-solo-machine-2026-07.md`
- Connector `AGENTS.md` — product carve, construction-order doctrine
- City-root `AGENTS.md` — Covenant, Triad, Hat, identity lineage,
 dashboard naming law (amendments 1–7)
- `workers/jiji/CONTRACT.md` — lane and obedience boundary
