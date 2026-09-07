# Connector — Join Config Template

**Issued by:** Connector neighborhood · **Ticket:**
**Template version:** 2026-08-11 · static fill-in (SEQUENCING Chunk 1)
**Design SoT:** [`docs/JOIN_DESIGN.md`](JOIN_DESIGN.md)
**Auth seam SoT:** [`docs/AUTH_ROAD.md`](AUTH_ROAD.md)
**Gate status:** G3 ✅ done · G5 open (Q2 — static template vs. `wl connect` CLI)

> **Mayor's note:** Fill in every `<PLACEHOLDER>` before handing this to the
> new citizen. Fields marked ⚠ are conditional — read the annotation before
> deciding whether to include them. Fields marked 🔲 depend on open gates —
> leave them as-is with their placeholder text for now.

---

## What this is

This sheet is the join kit the Mayor hands a new citizen the first time they
connect to the city. The citizen follows the four steps, pastes the MCP config
block into their environment, and bookmarks their door URLs. Nothing else is
required on their side.

**Shape TBD — SEQUENCING G5:** This static template is the recommended v1
form. Once You resolve Q2 (static template vs. `wl connect <host>`
CLI), this template may be superseded by or wrapped into a command. Until G5
clears, issue this sheet. (See `docs/SEQUENCING.md` Chunk 1.)

---

## Cover (Mayor fills in before handing over)

| Field | Value |
|---|---|
| **City folder** | `<CITY_FOLDER_PATH_OR_REPO_URL>` |
| **Handed to** | `<CITIZEN_DISPLAY_NAME>` |
| **Hand name assigned** | `<name>-<surface>` (e.g. `alex-desktop`) |
| **Issued by (Mayor)** | `<MAYOR_HAND_NAME>` |
| **Date issued** | `<YYYY-MM-DD>` |

---

## Step 1 — Get the Folder

Clone the city repos, or join a shared checkout on the same machine.

```
git clone <CITY_FOLDER_REPO_URL>
# — or —
# Ask the Mayor for the shared folder path and open it in your editor / terminal.
```

The folder is the city. `git` is the audit substrate; you do not need to know
that word to start working. Everything you need lives inside.

---

## Step 2 — Set Your Identity

Your **hand name** is `<name>-<surface>` — a stable, human-readable string
that matches your entry in `CITIZENS.md`. It is not an email or UUID.

Example: `alex-desktop` for Alex connecting from their desktop terminal,
`alex-phone-terminal` for a phone session.

You will use this string in your MCP config block (Step 3).

---

## Step 3 — Connect to the City Store

Paste the following block into your `~/.claude/settings.json` (under
`mcpServers`). Replace each placeholder with the value the Mayor gave you above.

### Local / trusted-LAN config (no auth token)

Use this when connecting on the same machine as the city host, or on a
trusted private network where the Mayor has not issued a bearer token.

```json
{
 "mcpServers": {
 "worklane": {
 "command": "wl",
 "args": ["mcp"],
 "env": {
 "WL_AGENT_ID": "<name>-<surface>",
 "WL_MCP_URL": "http://<HOST>:<PORT>"
 }
 }
 }
}
```

> **Solo / localhost:** omit `WL_MCP_URL` entirely (or set it to
> `http://localhost:<PORT>`). No token is needed — the localhost bypass
> applies automatically.

### ⚠ Networked config (with bearer token)

**Conditional: G4 / Q1 auth-tier decision** — include `WL_TOKEN` only if
the Mayor explicitly issued you a token. If the Mayor issued no token, use the
local config above regardless of whether you are connecting over the network.

> Gate G4 (You Q1 — trusted LAN vs. bearer enforcement) is **⬜ open**.
> Until this decision lands, the token field below is a placeholder only.
> Do not generate or paste a real token here until the Mayor provides one.

```json
{
 "mcpServers": {
 "worklane": {
 "command": "wl",
 "args": ["mcp"],
 "env": {
 "WL_AGENT_ID": "<name>-<surface>",
 "WL_TOKEN": "<BEARER_TOKEN_FROM_MAYOR>",
 "WL_MCP_URL": "http://<HOST>:<PORT>"
 }
 }
 }
}
```

`WL_TOKEN` is minted by the Mayor at invite time (city-law act). Never share
it; it identifies your hand to the city's auth surface. See
[`docs/AUTH_ROAD.md`](AUTH_ROAD.md) §3.2 for the full papers envelope design.

---

## Step 4 — Appear on the Roster

Once your MCP config is in place and your editor / terminal session restarts,
your hand (`<name>-<surface>`) is live on the city store. The Mayor adds your
entry to `CITIZENS.md` — you do not write this yourself.

Your `CITIZENS.md` entry (Mayor writes this for you):

```markdown
## <CITIZEN_DISPLAY_NAME>

- **Role:** Citizen
- **Git identity:** <Firstname Lastname> <email>
- **Hands:**
 - `<name>-<surface>` — <description: desktop / laptop / phone-terminal / etc.>
- **Since:** <YYYY-MM-DD>
```

The Mayor also adds you to `CITIZENS.md` in the city root as a Mayor-gated
act. You are not on the roster until this write happens.

---

## Door URLs — your city rooms

After connecting, the three city rooms are reachable here. Bookmark them.

| Room | URL | What it does |
|---|---|---|
| **Office** | `http://<HOST>:8796` | Projects (ProtocolCity) |
| **Desk** | `http://<HOST>:8799` | Tickets (WorkLane) |
| **Roster** | `http://<HOST>:8797` | Workers (WorkForce) |

> **🔲 G6 / Q3 open — Dashboard door shape TBD:** If the Mayor is running
> dashboards on the shared host, use the URLs above as-is. If each citizen
> runs local dashboard instances pointing at the shared store URL, substitute
> `localhost` for `<HOST>` and set `WL_MCP_URL` to the shared host in your
> MCP config. The Mayor will tell you which applies. (See `docs/SEQUENCING.md`
> G6 and `docs/JOIN_DESIGN.md` §6 Q3.)

Room headers display as `[Folder] Office`, `[Folder] Desk`, `[Folder] Roster`
(suite brand as subtitle). Browser tabs read the full
`ProtocolCity — Desk · Tickets` form.

---

## Ticket operations

Once the MCP is connected, tickets work identically to a local session. The
`WL_MCP_URL` env var points your `wl` / `wl_*` calls at the shared store
instead of localhost — no other change needed.

```
wl create "fix login flow" --project <neighborhood>
wl claim --project connector
wl close ...
```

Sign with your hand name (`WL_AGENT_ID`); the Desk will attribute your work
correctly once identity unhardcoding lands and the per-citizen identity slot is live.

> **Identity dependency:** Until WorkLane's identity unhardcoding work
> ships, a second hand connecting the MCP inherits the city-operator identity. All
> multi-citizen attribution waits on that gate. Connector's join flow is
> designed for the world after identity unhardcoding lands. (See `docs/JOIN_DESIGN.md` §2.)

---

## What to do if something is broken

1. **Can't reach the store URL:** check that the WorkLane server is running on
 the host; confirm `WL_MCP_URL` is set correctly in your MCP config.
2. **Identity showing as wrong user:** identity unhardcoding is still open; the Mayor can
 verify `WL_AGENT_ID` is in your config and restart your session.
3. **Token rejected (401):** ask the Mayor to re-mint; tokens are revoked when
 `CITIZENS.md` entries are removed.
4. **Room URL not loading:** confirm the correct door URL for your setup (shared
 host vs. local dashboards — see Q3 above).

File a ticket in the `connector` store (`project=connector`) for any join UX
issues. Engine-side auth errors belong in the `worklane` or `workforce` store.

---

## Sources (do not duplicate — link)

- [`docs/JOIN_DESIGN.md`](JOIN_DESIGN.md) — canonical join flow, carve boundary, open Qs Q1–Q3
- [`docs/AUTH_ROAD.md`](AUTH_ROAD.md) — §3.1 papers envelope; §3.2 citizen MCP config; §5 localhost bypass
- [`docs/SEQUENCING.md`](SEQUENCING.md) — gate checklist; Chunk 1 and G5; G4 and G6 open
- `ProtocolCity/docs/research/multi-citizen-design-2026-07.md` — CITIZENS.md schema §1, join flow §6
- City-root `AGENTS.md` — Covenant, Triad, dashboard naming law identity lineage
