# Connector — Road-Layer Auth Model

**Date:** 2026-07-27 · **Status:** design complete; implementation slices follow per engine neighborhood
**Author:** reed · Connector Desk

---

## What this doc is

The carve:

> *"Port auth enforcement stays on each engine (WorkLane, WorkForce, BluePrint suite)
> — connector does not own store or runner guts."*

This doc designs the **interface** at that seam: what "papers" a hand carries, what
each engine checks, how the solo city-operator bypass stays ergonomic, and what the
implementation follow-up tickets are. It does **not** implement auth code. That work
belongs in the owning engine's neighborhood.

**Substrate this doc builds on:**

- Covenant — citizen / hand / worker tier model
- Multi-citizen design — join flow, identity, transport section §6
- Beyond-solo-machine research — Track 1 auth gap analysis
- Identity unhardcoding — the hard blocker; designs here assume it landed
- Connector Join Design (JOIN_DESIGN.md) — the citizen-facing road this auth layer protects

---

## 1. Problem

Today the city runs on one laptop. Every port (WorkLane :8799, WorkForce :8797,
Office :8796) is localhost-only; network access IS the gate. `PROCESS §5.2` signs
every action with a stable identity — but that is **attribution**, not
**authentication**. Attribution answers *who claims to have done this*. It does not
stop a second machine from claiming to be `default-host-identity`, or a stranger on the
LAN from writing tickets as any registered citizen.

When the city goes multi-citizen on a shared host, that gap becomes real. The road
layer (connector) is the surface that carries credentials. The engine ports are the
checkpoints that verify them. This doc designs the interface; each engine owns its
checkpoint.

---

## 2. Threat model

Four contexts in ascending trust-difficulty, each calling for a different auth tier:

| Context | Who can reach the ports | Auth stance | Urgency |
|---|---|---|---|
| **Solo laptop (today)** | Only You, via localhost | Localhost trust — no auth needed | Done (default) |
| **Multi-citizen, trusted LAN** | A small team on a private network; no hostile actors assumed | Conventional identity; bearer tokens as step 2 if the team wants enforcement | v1 target |
| **Multi-citizen, WAN / untrusted** | Any machine that can reach the host IP | Per-citizen bearer tokens at a reverse proxy; workers carry service tokens | v2, You-gated |
| **Inter-city (horizon)** | Machines at other city installs, possibly unknown | Federation auth TBD — connector defines the paper format; inter-city trust is a separate gate | Not designed here |

**v1 target is trusted LAN.** The multi-citizen join flow names this
explicitly: "for a trusted-team LAN setup this is acceptable v1; bearer tokens are
the next step when the city goes WAN." This doc designs the bearer-token shape so
the upgrade path is clear, without requiring enforcement in v1.

---

## 3. Papers — what a hand carries

"Papers" is the auth credential the hand presents when connecting to a city port.
Connector defines the paper format. Engines verify it.

### 3.1 The envelope

A hand's papers at a given context are a small, opaque envelope:

```
{
 "identity": "<name>-<surface>", // WL_AGENT_ID — matches CITIZENS.md entry
 "token": "<bearer-token>", // absent in localhost / trusted-LAN solo mode
 "scope": "citizen" // "citizen" | "worker"
}
```

- **Identity** — already present (WL_AGENT_ID after identity unhardcoding lands). Human-readable,
 stable, matches the CITIZENS.md entry or PROCESS §5.2 worker row. Not a UUID.
- **Token** — a short-lived (or long-lived, You call) bearer token, minted at
 invite time for citizens and at employment for workers. Absent in localhost mode;
 present in any networked context that wants enforcement.
- **Scope** — `citizen` (a human hand, carries gate-clearing authority) or `worker`
 (an employed agent, authority ceiling is its CONTRACT). Distinct so engines can
 apply different checks without parsing identity strings.

### 3.2 Citizen papers

Minted at invite time by the Mayor (city-law act per §1). Carried in the
joining citizen's MCP config block alongside `WL_AGENT_ID`:

```json
{
 "mcpServers": {
 "worklane": {
 "command": "wl",
 "args": ["mcp"],
 "env": {
 "WL_AGENT_ID": "<name>-<surface>",
 "WL_TOKEN": "<bearer-token>",
 "WL_MCP_URL": "http://<host>:<port>"
 }
 }
 }
}
```

`WL_TOKEN` is absent in a solo-machine config (localhost bypass applies).

### 3.3 Worker service tokens

Workers carry a service token distinct from citizen tokens:

- Minted when the worker is employed (roster entry + PROCESS §5.2 row).
- Carried in the worker's `run.sh`, the same way credentials are fetched from the
 keychain today.
- Scoped to the worker's contract write surface: "file/update tickets in store X,
 nothing else." The written contract and the enforced token scope converge.
- The identity registry (PROCESS §5.2) IS the authentication subject list. The
 roster entry is the principal; the token is the credential.

### 3.4 Paper lifetime

| Kind | Minted by | Lives in | Revoked by |
|---|---|---|---|
| Citizen bearer token | Mayor at invite time | Citizen's MCP env / shell profile | Mayor removes CITIZENS.md entry + token invalidation (engine-side) |
| Worker service token | Employment flow (CONTRACT write) | Worker `run.sh` / keychain | De-employment: roster row removed, token invalidated |
| Localhost session | None — no token | Implicit | N/A (localhost bypass, see §5) |

Exact token format (JWT vs opaque short string vs HMAC) and rotation policy are
engine-side decisions, not connector's. Connector defines the envelope field name
(`WL_TOKEN`) and the minting ceremony trigger (invite / employment); the engine
decides the byte representation.

---

## 4. Engine interface — how engines check papers

Connector carries papers. Each engine owns its checkpoint. The interface between
them is a single check at the HTTP layer before any business logic runs:

```
incoming request
 → extract Authorization: Bearer <token> (or WL_TOKEN header, engine decides)
 → lookup identity claim (WL_AGENT_ID) in CITIZENS.md or PROCESS §5.2 registry
 → verify token matches the claim
 → allow / 401
```

### 4.1 WorkLane (:8799 — Desk)

WorkLane's HTTP server is the primary check surface. The auth middleware (a
follow-up ticket in the worklane store — do not implement here) intercepts every
MCP call and REST request before the TP router handles it.

**What connector provides:** `WL_AGENT_ID` + `WL_TOKEN` in the MCP config block
the citizen drops in. WorkLane reads both from the MCP `env` without connector
owning the server code path.

**What connector does NOT provide:** the middleware, the token store, the JWKS
endpoint, the 401 response — those are WorkLane's implementation.

### 4.2 WorkForce (:8797 — Roster)

WorkForce serves the Roster dashboard and dispatches workers. Same bearer-check
pattern. The worker's service token is carried by the daemon process; citizens
connect to Roster with citizen tokens.

**What connector provides:** the employment ceremony design (token minted with
the CONTRACT, carried in run.sh). WorkForce owns the enforcement.

### 4.3 Suite dashboards (Office :8796, Desk :8799, Roster :8797 — browser)

Browser rooms have two auth surfaces:

- **API calls** — same bearer token, passed as a cookie or auth header by the
 dashboard SPA (engine decision on the exact transport).
- **Page load** — a session cookie issued after initial token verification at the
 reverse proxy, or a per-dashboard auth wall.

**Recommended v1 for trusted LAN:** no dashboard auth wall. Network access is the
gate; the dashboards are read-only views that don't clear gates. Bearer enforcement
on the MCP/API surface is sufficient.

**Recommended v2 (WAN / untrusted):** a reverse proxy (nginx, Caddy) in front of
all three ports. The proxy checks the citizen's token before traffic reaches any
engine. No engine needs to reimplement the check if the proxy does it. Connector's
role: define the paper the proxy inspects (`WL_TOKEN` header, audience claim).

---

## 5. Localhost bypass

The solo city-operator must never need a token to use their own city. This is
**non-negotiable** — ergonomic bypass is a product constraint, not a security
tradeoff.

**Design rule (implemented per engine, not by connector):**

When `WL_MCP_URL` is absent or resolves to loopback (`localhost` / `127.0.0.1` /
`::1`), the engine treats the session as trusted and skips token verification.
`WL_AGENT_ID` is still read and used for attribution; only the token check is
bypassed.

**Connector's role:** document this rule in the join config template so citizens
know that a `WL_MCP_URL` pointing at a remote host activates token enforcement.
The bypass config (local) vs. the networked config (+ `WL_TOKEN`) are two distinct
join artifacts.

**Solo-laptop invariant:** a freshly founded city's default MCP config must not
include `WL_TOKEN`. The founding ceremony (`found` command) generates a
local-only config. Adding a token is an explicit Mayor act at multi-citizen invite
time.

---

## 6. Non-goals

These are out of scope for connector. They're listed here because they're the
natural wrong answers when reading this doc.

- **Connector does not implement auth enforcement.** No token store, no middleware,
 no JWKS, no 401 logic lives in this neighborhood. Filing an implementation ticket
 against connector for auth enforcement is a bug; the ticket belongs in worklane or
 workforce.
- **Connector is not a reverse proxy.** Setting up nginx or Caddy is an ops step for
 the host operator (the Mayor), not a shipped connector artifact. Connector may
 provide a reference config snippet; it does not own the proxy daemon.
- **Connector is not a Zapier/Composio-style pipe.** A connector worker that intakes
 email or CRM data is an *employed worker under contract* (Track 2), not an
 anonymous integration pipe. The auth surface for intake workers is the service
 token (§3.3), not connector-the-product.
- **Connector is not a fifth room.** It is the road, not a destination. It has no
 dashboard of its own; citizens never "open Connector." Its artifacts are config
 blocks and invite docs.
- **Inter-city federation is not designed here.** The paper envelope (§3.1) names
 the shape a future inter-city auth could inhabit; the trust model between city
 installs is a separate gate.

---

## 7. Dependencies and sequencing

| Dependency | Status | What connector waits on identity unhardcoding |
|---|---|---|
| — identity unhardcoding | Open (hard blocker) | `WL_AGENT_ID` per-citizen config; neutral fallback `you`; PROCESS §5.2 citizen-hand rows. Everything in this doc that names `WL_AGENT_ID` assumes identity unhardcoding shipped. |
| — multi-citizen design | Design complete | Defines CITIZENS.md, the join flow, transport topology. AUTH_ROAD.md is the auth layer on top of that transport. |
| **WorkLane flip** (public) | Pending | Network auth implementation in any engine is gated until WorkLane flips public. Design is free now. |
| **Auth tier decision** (Q1 in JOIN_DESIGN.md) | You-gated | Trusted LAN vs. bearer enforcement from day one. This doc designs both; which ships first is a You call. |

**Sequencing rule:** design is free now. Implementation of any token check in any
engine is an un-table decision You make after the WorkLane flip and when
a real networked city is being provisioned. Do not implement ahead of that gate.

---

## 8. Implementation follow-ups

These are the concrete next tickets, cross-filed by engine. Do not implement from
here; route after the gating decisions resolve.

| Neighborhood | Scope | Depends on |
|---|---|---|
| **worklane** | Auth middleware slice: read `WL_TOKEN` header, verify against PROCESS §5.2 token store, 401 on mismatch; localhost bypass rule | identity unhardcoding, WorkLane flip, Q1 auth decision |
| **worklane** | Token minting ceremony: issue citizen token at invite time; rotate on revocation | Same gates + CITIZENS.md write flow |
| **worklane** | Worker service token: mint at employment (PROCESS §5.2 row write), store in keychain; carried in run.sh | Auth middleware slice |
| **workforce** | Auth check on :8797 (Roster / WorkForce HTTP surface) | worklane auth slice pattern established first |
| **connector** | Join config template: two variants — local (no token) + networked (+ `WL_TOKEN`) — **✅ delivered** 2026-08-11 · [`docs/JOIN_CONFIG_TEMPLATE.md`](JOIN_CONFIG_TEMPLATE.md) | |
| **connector** | Invite artifact: token generation step for Mayor; guidance on reverse proxy for WAN contexts | Q1 auth decision |
| **ProtocolCity** | FOUNDING.md: "Inviting a citizen (networked)" section — how to stand up a shared host, where the proxy goes | This doc |

---

## Sources

- `connector/docs/JOIN_DESIGN.md` — canonical join flow, carve boundary, open questions Q1–Q3
- `ProtocolCity/docs/research/multi-citizen-design-2026-07.md` — transport §6, identity §2, citizens registry §1
- `ProtocolCity/docs/research/beyond-solo-machine-2026-07.md` — Track 1 auth gap; worker service token shape; roster-as-principal-list
- Connector `AGENTS.md` — product carve, construction-order doctrine
- City-root `AGENTS.md` — Covenant, Triad lineage, PROCESS §5.2 identity registry
- `workers/jiji/CONTRACT.md` — lane boundary (never touch live credentials; never implement auth ahead of gates)
