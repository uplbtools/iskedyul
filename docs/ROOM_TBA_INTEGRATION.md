# Room TBA integration

How Iskedyul reads campus data from [Room TBA](https://github.com/uplbtools/room-tba) and links back into the map app.

**Base URLs**

| Environment | Origin |
| ----------- | ------ |
| Production | `https://room-tba.uplbtools.me` |
| Staging | `https://staging.room-tba.uplbtools.me` |
| Local dev | `http://localhost:4321` |

All endpoints below are **read-only GET** unless noted. No API key for public reads today.

---

## Endpoints Iskedyul needs

### `GET /api/terms`

Academic terms with class counts. Pick the active term for planning.

```json
[
  {
    "id": 1252,
    "label": "2nd Sem AY 2025–2026",
    "startDate": "2026-01-05",
    "endDate": "2026-05-22",
    "classCount": 3124
  }
]
```

**CRS note:** `terms.id` matches AMIS/CRS `term_id` (1251 = 1st sem, 1252 = 2nd sem, 1253 = midyear).

### `GET /api/classes`

Primary data source for section picker and grid population.

| Query param | Purpose |
| ----------- | ------- |
| `term_id` | Filter to one term (required for planner) |
| `course_code` | Prefix search, e.g. `MATH` → paginated |
| `limit` / `offset` | Pagination when searching |
| `room_code` | All classes in a room (not used by planner MVP) |

**Bulk load (MVP):** `GET /api/classes?term_id=1252` — full array for conflict matching client-side.

**Search (post-MVP):** `GET /api/classes?term_id=1252&course_code=MATH%2027&limit=25&offset=0`

```json
{
  "rows": [
    {
      "id": 88421,
      "courseCode": "MATH 27",
      "section": "AB-1L",
      "type": "LAB",
      "schedule": ["T 1-4", "Th 1-4"],
      "roomCode": "PSLH-A",
      "roomId": 412,
      "courseTitle": "Analytic Geometry",
      "termId": 1252,
      "directions": "..."
    }
  ],
  "total": 3
}
```

**Schedule slot format:** strings like `MWF 8-9`, `T 1-4`, `MTWTHFS 7-12` — parse with shared logic in `packages/planner-core` (mirror Room TBA's `schedule-import/day-stops.ts` conventions).

**Rows without rooms:** `roomCode: null` for thesis/special-problem types — show in grid but link as "Room TBA" with explainer copy from Room TBA's scheduled-types policy.

### Deep links into Room TBA

| Intent | URL pattern |
| ------ | ----------- |
| Room on map | `https://room-tba.uplbtools.me/?q={roomCode}` |
| Course finals | `https://room-tba.uplbtools.me/?q={courseCode}` (existing search) |
| Building | `https://room-tba.uplbtools.me/?q={buildingName}` |

Future: `?plan={shareId}` if Room TBA adds schedule-route handoff — tracked in both repos.

---

## Schedule import parity

Room TBA already parses CRS/AMIS exports in `src/lib/schedule-import/`:

- `parse-import.ts` — JSON/text → normalized rows
- `match-classes.ts` — rows ↔ `ClassMapValue` by `(courseCode, section, type)`
- `day-stops.ts` — order stops for walking route

**Iskedyul options (pick in Phase 1):**

1. **Extract** `schedule-parse` into a shared npm workspace consumed by both repos (cleanest long-term).
2. **Copy** normalizer + tests once, document drift risk (fastest MVP).
3. **API delegate** — new Room TBA route `POST /api/schedule/parse` (adds server coupling).

Recommendation: **(2) for MVP**, open Room TBA issue to extract **(1)** when planner stabilizes.

Normalized row shape (no instructor PII):

```ts
type ImportedScheduleRow = {
  courseCode: string;
  section: string;
  type: string;
  schedule: string[];
};
```

---

## Data Iskedyul does *not* duplicate

Keep Room TBA authoritative for:

- Room/building coordinates and aliases
- Map tiles, jeepney routes, events
- Finals exam table (`final_exams` — future overlay via new read API or export)
- AMIS import pipeline and term metadata fixes

Iskedyul stores only **user plan state** (selected class IDs, draft names) locally.

---

## CORS and caching

Room TBA API routes are same-origin today. For Iskedyul on a **different subdomain** (e.g. `iskedyul.uplbtools.me`):

- Room TBA must expose `Access-Control-Allow-Origin` on `/api/classes` and `/api/terms` (already on some routes like `class-counts`).
- Client: cache term + class snapshot in IndexedDB with `syncKey` or ETag when Room TBA adds it ([#TBD issue in room-tba]).

Until CORS is wired, MVP can proxy via Iskedyul server route (`/api/room-tba/classes`) to avoid browser blocks.

---

## Version coupling

| Iskedyul release | Expects Room TBA |
| ---------------- | ---------------- |
| MVP | `/api/terms`, `/api/classes?term_id=` stable `ClassMapValue` shape |
| v0.2 | Finals read API or shared finals JSON |
| v0.3 | Optional `POST /api/planner/export` for handoff |

Breaking changes to `ClassMapValue` require coordinated releases — document in Room TBA CHANGELOG.

---

## Open coordination issues

File in **room-tba** when implementation starts:

1. CORS headers on public class/term routes
2. Extract `schedule-import` package or publish `@uplbtools/schedule-parse`
3. Finals exam JSON endpoint for planner overlay
4. Shared `term_id` default helper (which term is "current")

File in **iskedyul**:

1. Proxy vs direct fetch decision after CORS spike
2. Slot parser test vectors copied from Room TBA fixtures
