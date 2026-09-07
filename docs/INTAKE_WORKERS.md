# Connector — Intake Workers Design

**Date:** 2026-07-27 · **Status:** design complete;
implementation gated on WorkLane flip + identity unhardcoding + You hire decision
**Author:** reed · Connector Desk

---

## What this doc is

Beyond-solo-machine Track 2 names a doctrine: **an integration is an
employee, not a plugin.** This doc captures that doctrine as neighborhood
design — the vocabulary split, the CONTRACT template requirements, the Roster
appearance, and the first proving ground — so the pattern is ready when
implementation gates open.

No real intake worker is hired here. No OAuth credentials, no live external
MCP configs, no network code.

**Substrate this doc builds on:**

- Covenant — citizen / hand / worker tier model
- Beyond-solo-machine research — Track 2: connector workers as intake
 employees; CONTRACT sketch; Outlook proving ground
- Auth road (AUTH_ROAD.md) — §3.3 worker service tokens; §6 non-goals
 (connector is not a pipe; intake workers carry service tokens)
- connector/AGENTS.md — product carve; construction-order doctrine
- Socials press-pass pattern (city-root AGENTS.md) — the write direction that
 intake workers invert

---

## 1. Vocabulary split

The word "connector" carries two meanings in this city. This section is the
canonical disambiguation.

| Term | What it is | Owned by |
|---|---|---|
| **Connector** (capitalized, the product) | The road layer — join UX + client connect. A citizen walks this road to reach a city hosted on a network. Not a room, not a dashboard, not a pipe. | `connector/` neighborhood |
| **intake worker** (e.g. `intake-outlook`) | An employed agent whose contract scopes it to reading one external MCP and writing tickets. A worker on the Roster, not a product feature. | WorkForce roster + `connector/workers/` CONTRACT templates |

**Rule for copy and tickets:** when a ticket or doc says "connector" without
context, it means the product (the road). When naming a specific intake
integration, say "intake worker" or name the worker (`intake-outlook`). Never
write "a connector" to mean an integration pipe.

**Why the ambiguity is real:** connector-the-product carries papers for humans
joining the city; an intake-worker carries data from an external source into
the city's ticket store. Both cross a boundary; only the intake worker is
employed. The vocabulary split keeps the carve clean.

---

## 2. The doctrine

**An integration is an employee, not a plugin.**

An intake worker is architecturally the `socials` worker with the read/write
direction inverted:

| | Reads | Writes |
|---|---|---|
| **Socials worker** | City repos + ticket history (press pass) | Nowhere inside the city (drafts to `socials/` only) |
| **Intake worker** | One external MCP only | Tickets in one named store only |

Both appear on the Roster. Both sign everything. Both operate under a written,
readable contract any citizen can inspect. Neither is a Zapier/Composio-style
pipe — anonymous, unaudited, stateless.

**Why this beats a pipe.** The 2026 market already has Outlook→anything MCP
connectors (Composio, StackOne, Merge, Zapier MCP, Copilot Studio —
landscape). They are pipes. The three things an employed worker adds:

1. **Visibility.** The worker appears on the Roster with a shift and a ledger;
 a pipe is invisible until it breaks.
2. **Accountability.** Every ticket filed is signed with the worker's identity
 and traces back to the source; a pipe's output is anonymous.
3. **Enforceability.** The contract names what the worker may touch; the
 service token is scoped to match; written rule and enforced rule converge.

The intake worker is not "we also have integrations." It is "your integrations
are governed employees like everyone else in the city."

---

## 3. CONTRACT template requirements

Every intake worker's `CONTRACT.md` must satisfy this checklist. These are the
four behaviors that separate a governed intake worker from an anonymous pipe.
The placement in the adoption kit is a You
call; this section is the design record.

### 3.1 Provenance

Every ticket the worker files records its source. Minimum required fields on
the ticket body or in a standard opening comment:

```
source: <connector-id> # e.g. outlook, crm-salesforce
external_id: <source-system-id> # message-id, record-id, or permalink
external_url: <link> # optional but preferred; lets a citizen trace back
```

The worker records these at create time. A citizen following up should always
be able to reach the original email, record, or event from the ticket.

**CONTRACT clause:** `This worker records source, external_id, and external_url
on every ticket it files.`

### 3.2 Dedupe / idempotency

The worker keys on `external_id`. Required behavior:

- `external_id` not seen → **create** a new ticket.
- `external_id` already has an **open** ticket → **update** (add a comment;
 never file a duplicate).
- `external_id` already has a **closed** ticket → **create** a new ticket (the
 source sent a new, independent signal).

This is the single hardest part of any inbox→ticket integration and the main
reason a contract beats a naive pipe. The worker's CONTRACT must name the exact
field it keys on and which store it searches before filing.

**CONTRACT clause:** `This worker dedupes on external_id. Before filing, it
searches store <name> for an open ticket with the same external_id; if found,
it updates; it never duplicates.`

### 3.3 Label namespace `intake:*`

A reserved label namespace per intake worker: `intake:<source>`.

Examples:

- `intake:outlook` — all tickets from the Outlook intake worker
- `intake:crm-salesforce` — all tickets from a hypothetical Salesforce worker

The worker's CONTRACT names its namespace and uses only labels within it. This
keeps intake filterable and makes the worker's write scope enforceable (the
service token can be scoped to labels starting with `intake:<source>`).

**CONTRACT clause:** `This worker labels every ticket with intake:<source>.
It does not use labels outside that namespace.`

### 3.4 Write-only-to-tickets

The inverse of the socials press pass. An intake worker's authority ceiling:

| Surface | Allowed |
|---|---|
| One external MCP (read) | ✅ |
| Tickets in one named store (create / update / comment) | ✅ |
| Code, config, or any file in any repo | ❌ |
| Other ticket stores | ❌ |
| Gate-clearing (any `Needs You ·` act) | ❌ — `scope: worker`, no gate authority |
| External systems other than its assigned MCP | ❌ |

**CONTRACT clause:** `This worker reads only <external-mcp-name> and writes
only tickets in store <name>. It touches no files, no other stores, and clears
no gates.`

---

## 4. Roster appearance

An intake worker appears on the WorkForce Roster exactly as any other employed
worker. The worker tier already has this shape; no new UI is needed for intake
workers specifically.

| Roster field | Intake-worker value |
|---|---|
| **Identity** | `intake-<source>` (e.g. `intake-outlook`) — matches PROCESS §5.2 row |
| **Kind** | `lane` (scoped to one store + one label namespace) |
| **Shift** | Daemon-run or scheduled-run; shift start / end recorded normally |
| **Ledger** | Every create / update action on a ticket is a ledger entry; this log is the provenance trail |
| **Contract** | `connector/workers/<name>/CONTRACT.md` — readable by any citizen |

The Roster shows the worker as hired capacity, not a config toggle. A citizen
can see when it last ran, what it filed, and what its contract says — from the
same dashboard used for every other worker.

**Service token** (AUTH_ROAD.md §3.3): minted when the worker is employed;
carried in `run.sh` / keychain; scoped to `file/update tickets in store X with
labels intake:<source>`. The written contract and the token scope converge.

---

## 5. First proving ground: Outlook intake

 names Outlook as the candidate for the first real intake worker.

**Why Outlook:**
- The external MCP surface exists (Composio, StackOne, native Graph API MCP) —
 the read side is solved in the market.
- Email→ticket is the clearest "external event becomes city work item."
- The dedupe challenge (same email thread → one ticket, not N tickets) is
 concretely testable against the checklist in §3.2.

**What a proving-ground run would verify:**

1. CONTRACT template sufficient — can a worker be employed using only the §3
 checklist, or does it need additions?
2. Dedupe logic holds — thread-id as `external_id`; update-not-duplicate
 across multiple runs on the same inbox state.
3. Roster visibility works as designed — a citizen finds the worker, reads its
 contract, and sees its ledger without any special UI surface.
4. Vocabulary split holds in practice — "Outlook intake worker" is unambiguous
 in tickets; "Connector" is never confused with this worker.

**Out of scope here:** the actual employment (hire), OAuth credential
management, MCP config for the Graph API. Those are implementation tickets
gated on: stable CONTRACT template, service-token design (AUTH_ROAD.md §3.3),
WorkLane flip, and a You green-light on the external MCP dependency.

---

## 6. Sequencing + gates

| Dependency | Status | Blocks |
|---|---|---|
| **AUTH_ROAD.md §3.3** — worker service tokens | Design complete | Contract checklist aligns; token minting implementation gated on WL flip + identity unhardcoding |
| **WorkForce Roster** | Running | Roster appearance is already supported; no new UI needed for intake workers |
| **Track 2 doctrine adoption** | You-gated | Doctrine recorded here; formal adoption is a You call; this doc makes it actionable |
| **FOUNDING.md** — adoption kit update | Open | CONTRACT template in §3 belongs in the kit; routing is a You call |
| **First real hire (Outlook)** | Not yet — gated | Requires stable CONTRACT template (this doc), service token (post-WL flip), You green-light on external MCP dependency |

**Design is free now.** The CONTRACT template, vocabulary split, and Roster
appearance are ready to use when implementation gates open.

---

## Non-goals

- **Connector does not implement the intake worker's MCP calls.** The Outlook
 MCP surface is the external vendor's responsibility; connector owns the
 employment pattern, not the transport to Graph API.
- **Connector is not a Zapier-style pipe.** An intake worker is an employee;
 the pipe is the anti-pattern (§2).
- **No worker is hired in this doc.** This is a design record, not an
 employment slip.
- **No credentials or secrets land here.** OAuth tokens, API keys, and service
 tokens stay in the keychain and `run.sh` — never in git.

---

## Sources

- `ProtocolCity/docs/research/beyond-solo-machine-2026-07.md` — Track 2
 doctrine; CONTRACT sketch; Outlook proving ground; pipe landscape (Composio
 et al.)
- `connector/docs/AUTH_ROAD.md` — §3.3 worker service tokens; §6
 non-goals (connector is not a pipe; workers carry service tokens)
- `connector/docs/JOIN_DESIGN.md` — citizen-vs-worker identity tiers;
 carve boundary
- `connector/AGENTS.md` — product definition; construction-order
 doctrine; beyond-solo Track 2 reference
- City-root `AGENTS.md` — Covenant;
 socials press-pass pattern (read-everywhere / write-nowhere — the inverse
 that intake workers hold)
- `workers/jiji/CONTRACT.md` — lane boundary: never hire a real worker this
 shift; never touch live credentials or OAuth secrets
