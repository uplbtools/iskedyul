# Iskedyul

**Build your UPLB semester timetable: then see it on the campus map.**

A course planner for [UPLB](https://uplb.edu.ph) students: search offerings, pick LEC/LAB sections, spot schedule clashes, and hand off to [Room TBA](https://room-tba.uplbtools.me) for room locations, walking routes, and finals info.

> Status: **early planning**: see [docs/IMPLEMENTATION_PLAN.md](docs/IMPLEMENTATION_PLAN.md).

## Why this exists

CRS tells you *what* you enrolled in. Room TBA tells you *where* it meets. Nothing yet ties them into a **visual week view** you can iterate on before enlistment: or export after.

Iskedyul fills that gap:

- **Plan before you commit**: draft timetables, compare section options, check conflicts
- **Room-aware**: every block links to the room on the campus map
- **Same data as Room TBA**: class rows come from the shared AMIS import pipeline, not a stale spreadsheet
- **No account required**: plans live in the browser (localStorage) until we add optional sync

## Core flows

```text
Pick term → Search course (e.g. MATH 27) → Add LEC + LAB sections
 ↓
Weekly grid highlights clashes in red
 ↓
"Open in Room TBA" → map pin, building directions, day route between classes
 ↓
"Export to Google Calendar" → download .ics → import in Google Calendar
```

### Import path

Already enlisted? Paste or upload your CRS/AMIS schedule export: same normalized format Room TBA uses for schedule import: and Iskedyul builds the grid, then matches rows to campus rooms.

## Relationship to Room TBA

| Iskedyul | Room TBA |
| -------- | -------- |
| Timetable builder & conflict checker | Campus map & room search |
| "Which sections fit my week?" | "Where is PSLH-A?" |
| Draft plans (multiple alternatives) | Live schedules, finals, events |
| Consumes public class API | Source of truth for rooms & classes |

Deep integration spec: [docs/ROOM_TBA_INTEGRATION.md](docs/ROOM_TBA_INTEGRATION.md).

## Planned features

| Feature | MVP | Later |
| ------- | --- | ----- |
| Weekly grid (M–Sat) | ✓ | |
| Course search + section picker | ✓ | |
| Time conflict detection | ✓ | |
| Room TBA deep links | ✓ | |
| AMIS/CRS import | ✓ | |
| **Export to Google Calendar** (`.ics` download) | | ✓ |
| Multiple saved plans | | ✓ |
| Walking time between classes | | ✓ (reuse Room TBA route logic) |
| Final exam overlay | | ✓ |
| GE / free-elective checklist | | ✓ |

Calendar export detail: [docs/GOOGLE_CALENDAR_EXPORT.md](docs/GOOGLE_CALENDAR_EXPORT.md).

## Repo layout (planned)

```text
iskedyul/
├── apps/
│ └── web/ # planner UI (mobile-first)
├── packages/
│ ├── planner-core/ # grid, conflicts, slot parsing
│ ├── room-tba-client/ # typed fetch wrapper for Room TBA APIs
│ └── schedule-parse/ # shared import normalizer (may extract from room-tba)
└── docs/
 ├── IMPLEMENTATION_PLAN.md
 ├── ROOM_TBA_INTEGRATION.md
 └── GOOGLE_CALENDAR_EXPORT.md
```

## Contributing

Planning phase: game-changing feedback welcome via issues. Code contributions open after Phase 0 spike.

## License

MIT: see [LICENSE](LICENSE).
