# Connector — Project instructions (L1 CORE)

**Public brand:** **Connector** (ratified 2026-07-27 — same string as
the folder/store; no second marketing name).
**Wire:** store `connector` · prefix **`conn-`** · `project=connector`
**Package intent (not shipped):** `protocolcity-connector`
**Public face:** [protocolcity/Connector](https://github.com/protocolcity/Connector)
**Workspace L0:** city CORE AGENTS · always-work:
`docs/specs/ALWAYS_WORK_PROCESS.md` at workspace root

Fourth suite product carve: the **road layer** — host a city folder
on the network and connect a citizen (You) to it. Join UX + client connect
live here. Port auth stays on each engine. Not a fifth suite room. Not a
Zapier-style pipe (later integrations = employees under contract).

Construction-order: **rooms → buildings → roads.** Solo-laptop v1
does **not** require this product to ship. Design now; network auth
implementation waits on [`docs/SEQUENCING.md`](docs/SEQUENCING.md) gates.

Vendor pointers (`CLAUDE.md` / `GROK.md`) → `@AGENTS.md` only.

## Non-negotiables

1. **No secret leak.** Credentials, tokens, and live host secrets never land in git.
2. **One work order per project** when work crosses engines — join UX here;
   engine auth on WorkLane / WorkForce tickets.
3. **Public brand is Connector** — do not invent Roadway/Transit/etc. as the
   product name; road/transit language may appear in copy only.
4. **Posting / marketing** of Connector is city-operator-only until ship pressure.

## Desk and land

- Every change ties to a `conn-*` work order. Lifecycle:
  suite WorkLane PROCESS §5.
- Discover actual WorkForce capacity before routing. No Connector lane is
  currently registered. Jiji, Reed and Zach papers are historical. Authorized
  host documentation work uses `worker:you` + `you:host`.
- **Git:** public face is **protocolcity/Connector**. Land finishing slices on
  local `main` (ff-only from the shift worktree into the primary checkout);
  cite that SHA in Links before any public push.
- Papers are Markdown. Named lines of work live in
  [`PROGRAMS.md`](PROGRAMS.md). There is no running product binary; `docs/`
  is the design vault.

## What to read

| Need | Doc |
|---|---|
| Layers / SoT / invariants | [`ARCHITECTURE.md`](ARCHITECTURE.md) |
| Named programs + gate state | [`PROGRAMS.md`](PROGRAMS.md) |
| When code is allowed | [`docs/SEQUENCING.md`](docs/SEQUENCING.md) |
| Join UX design | [`docs/JOIN_DESIGN.md`](docs/JOIN_DESIGN.md) |
| Auth seam (engines enforce) | [`docs/AUTH_ROAD.md`](docs/AUTH_ROAD.md) |
| Intake workers as employees | [`docs/INTAKE_WORKERS.md`](docs/INTAKE_WORKERS.md) |
| Historical contract | [`workers/jiji/CONTRACT.md`](workers/jiji/CONTRACT.md) |

## Tree (short)

| Path | Role |
|---|---|
| `AGENTS.md` | This CORE |
| `ARCHITECTURE.md` | Binding structure |
| `PROGRAMS.md` | Named lines of work |
| `docs/` | Design vault (several **DITCHED**) |
| `workers/jiji/` | Historical lane papers |
| `prototypes/` · `archive/` | Retired — not product |

Do not hand-edit the generated hands block (workspace jobs leak is doctor /
sibling ProtocolCity scope).

<!-- bp:generated:hands -->
<!-- /bp:generated:hands -->
