# Iskedyul Agent Guide

**Human developers:** start with [README.md](README.md) and [docs/IMPLEMENTATION_PLAN.md](docs/IMPLEMENTATION_PLAN.md).

Org-wide agent defaults: see [room-tba/AGENTS.md](https://github.com/uplbtools/room-tba/blob/main/AGENTS.md). This file tailors that playbook to **iskedyul** (planning phase).

## Status

**Planning repo: no `apps/*` packages yet.** Read [docs/ROOM_TBA_INTEGRATION.md](docs/ROOM_TBA_INTEGRATION.md) before designing APIs or data shapes.

## Doc map

| When | Read |
| --- | --- |
| Product scope | [docs/IMPLEMENTATION_PLAN.md](docs/IMPLEMENTATION_PLAN.md) |
| Room TBA class/room API contract | [docs/ROOM_TBA_INTEGRATION.md](docs/ROOM_TBA_INTEGRATION.md) |
| AMIS term semantics (consumer) | [room-tba AGENTS.md § AMIS](https://github.com/uplbtools/room-tba/blob/main/AGENTS.md) |

## Planned stack (target: not all implemented)

- **Monorepo:** npm workspaces
- **Runtime:** Bun + **Svelte 5** + TypeScript
- **App shell:** SvelteKit or Astro + Svelte (TBD in plan)
- **Deploy:** Vercel
- **MVP storage:** localStorage: no server DB in v1
- **Export:** Google Calendar `.ics` generation client-side

Follow **Room TBA** conventions for community links, maroon accent UI, and calm static UX when UI work starts.

## Branches and deploy

- **Default branch:** `main`
- Adopt **`staging` → `main`** before public launch (match room-tba ship pipeline)

## Room TBA integration rules

- **Do not re-import AMIS** in iskedyul: consume Room TBA's `/api` class and room data
- **CRS `term_id` semantics** match room-tba (1251 / 1252 / 1253): never guess term type from row counts
- Cache Room TBA responses in localStorage with explicit sync/version keys
- Offline-first timetable editing should work when Room TBA bootstrap is cached (future)

## Verify before done (when code exists)

| Step | When |
| --- | --- |
| `bun run check` / app lint+test | Once real packages exist |
| `bun run build` | Before PR merge |
| Layout @ 320px | Timetable grid must not overflow: mirror room-tba mobile rules |

## How to work

- **Bias toward action** on integration spikes; ask when Room TBA API shape is ambiguous
- **Same PR:** feature + tests + doc updates to `ROOM_TBA_INTEGRATION.md` when contracts change
- **`gh` by default** for cross-repo issues (link room-tba `#NNN` when API gaps block you)

## Commits

- Conventional Commits: `docs: …`, `feat(web): …`, `fix(api): …`
