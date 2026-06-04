# REST API

`cad-planner-server` exposes a small REST surface on
`https://cad.openie.dev/api`. The bulk of the substrate's
capability lives behind the [MCP endpoint](./mcp-tools.md); the
REST surface is mostly health/session/artifact bookkeeping plus
a few specialised handoff routes.

## Endpoints

| Method | Path | Purpose |
|---|---|---|
| `GET`  | `/api/health` | Liveness probe — returns 200 when the server is up. |
| `POST` | `/api/plan` | Submit a planning request; returns a transcript of agent turns. |
| `POST` | `/api/napkin-plan` | Same as `/api/plan` but with the napkin-tailored tool subset. |
| `GET`  | `/api/sessions/{id}` | Inspect a session's project + agent history. |
| `GET`  | `/api/sessions/{id}/joules` | Per-session "joules" accumulator (per tool call). |
| `GET`  | `/api/sessions/{id}/pcb.kicad_pcb` | The live PCB of an MCP session as a `.kicad_pcb` text blob. KiCanvas-compatible. |
| `GET`  | `/api/joules` | Server-wide joules totals. |
| `POST` | `/api/handoff` | Create a handoff blob (cross-process project transfer). |
| `GET`  | `/api/handoff/{id}` | Fetch a handoff blob. |
| `POST` | `/api/design-pcb-from-chip` | Specialised one-shot chip→PCB design call. |
| `POST` | `/api/design-pcb-from-chip-with-layout` | Same with an explicit layout. |
| `GET`  | `/api/artifact/{id}` | Fetch an artifact (STEP / STL / Gerber / etc.). |
| `GET`  | `/api/artifacts` | List artifacts. |
| `GET`  | `/api/kicad-httplib/v1/` | Roots of the KiCad HTTP-lib catalogue endpoint. |
| `GET`  | `/api/kicad-httplib/v1/categories.json` | Categories list. |
| `GET`  | `/viewer/kicad` | Interactive KiCanvas-based PCB viewer. Accepts `?src=`, `?artifact=`, or `?session=`. |
| `GET`/`POST` | `/mcp` | JSON-RPC endpoint (POST) + landing page (GET). See [MCP tool reference](./mcp-tools.md). |

## Sessions

The same session id covers both REST and MCP. After any
`POST /mcp` call (or `POST /api/plan`) the response header
`Mcp-Session-Id: <uuid>` identifies the per-process state. Re-use it
on subsequent calls — REST or MCP — to continue working on the same
project.

## Authentication

Some POST endpoints require a Bearer token configured at server start
time (see the substrate's deploy docs). Public read endpoints
(`/api/health`, `/api/artifact/{id}` on artifacts owned by the
caller's session, `/viewer/*`) work without auth.

## Why REST is thin

MCP is the substrate's primary surface — every meaningful capability
ships there. REST exists for orchestration (planning, session
introspection, artifact fetch, viewer rendering) and for tools that
don't yet speak MCP. Adding more REST endpoints isn't on the roadmap;
new capabilities ship as MCP tools.
