# Google Calendar export

How Iskedyul turns a semester plan into events students can add to **Google Calendar** (and any other app that reads `.ics`).

**Status:** planned for **Phase 2** (after MVP grid). Generator lives in `packages/planner-core`; UI in `apps/web`.

---

## User-facing flows

### Primary: download `.ics` → import to Google Calendar

Works on desktop and mobile without Google sign-in or OAuth.

1. Student finishes a plan and taps **Export calendar**.
2. Browser downloads `iskedyul-{termId}-{planName}.ics`.
3. Google Calendar → **Settings** → **Import & export** → **Import** → select file → choose target calendar.

Short in-app hint after download:

> Open Google Calendar on a computer to import, or forward the file to yourself and open it on mobile.

### Secondary: open Google Calendar (single event)

For one-off "add this block" from the grid context menu:

```
https://calendar.google.com/calendar/render?action=TEMPLATE
  &text={encodedTitle}
  &dates={startUtc}/{endUtc}
  &location={encodedRoom}
  &details={encodedDescription}
```

Limited to **one occurrence**: use for debugging or sharing a single meeting, not full-semester export.

### Out of scope (for now)

- **Google Calendar API + OAuth**: full two-way sync; needs accounts, token storage, and ToS review.
- **Live `webcal://` subscription URL**: requires hosted feed endpoint and cache invalidation when Room TBA re-imports classes.

Revisit subscription feeds if students ask for auto-updates when room assignments change mid-semester.

---

## Event model

Each **grid block** (one day + time range for one class section) becomes one **recurring** calendar event for the term.

| ICS field | Source |
| --------- | ------ |
| `SUMMARY` | `{courseCode} {type} ({section})` e.g. `MATH 27 LAB (AB-1L)` |
| `DESCRIPTION` | `{courseTitle}\nSection: …\nRoom: …\nhttps://room-tba.uplbtools.me/?q={roomCode}` |
| `LOCATION` | `{roomCode}` or `Room TBA` if null |
| `DTSTART` / `DTEND` | First occurrence in **Asia/Manila** (`TZID=Asia/Manila`) |
| `RRULE` | `FREQ=WEEKLY;BYDAY={MO,TU,...};UNTIL={termEndUtc}` |
| `UID` | `{planId}-{classId}-{day}@iskedyul.uplbtools.me` (stable for re-export) |
| `URL` | Room TBA deep link when `roomCode` present |

**Term bounds:** from `/api/terms`: `startDate` / `endDate` set `RRULE UNTIL` and clip the first week if enlistment starts mid-week.

**Clashing sections:** export **all** added classes; do not hide clashes. Description may prefix `⚠ Overlaps with another class` when `planner-core` reports a conflict (optional v0.2 polish).

**Finals:** separate `VEVENT` series when finals data is available (Phase 3): one-off timed events, no `RRULE`.

---

## Timezone and slot mapping

UPLB schedules use **Philippine time**. All exported datetimes use `VTIMEZONE` for `Asia/Manila` (no DST).

Slot parsing (shared with grid):

| Schedule string | Maps to |
| --------------- | ------- |
| `MWF 8-9` | Mon/Wed/Fri 08:00–09:00 |
| `T 1-4` | Tue 13:00–16:00 |
| `Th 10-12` | Thu 10:00–12:00 |

Hour interpretation matches Room TBA / AMIS convention (7–21 = 7 AM–9 PM). Document edge cases in `planner-core` tests.

**Example RRULE** for `MWF 8-9`, term ending 2026-05-22:

```ics
BEGIN:VEVENT
UID:plan1-88421-M@iskedyul.uplbtools.me
DTSTAMP:20260703T120000Z
DTSTART;TZID=Asia/Manila:20260106T080000
DTEND;TZID=Asia/Manila:20260106T090000
RRULE:FREQ=WEEKLY;BYDAY=MO,WE,FR;UNTIL=20260522T155959Z
SUMMARY:MATH 27 LAB (AB-1L)
LOCATION:PSLH-A
DESCRIPTION:Analytic Geometry\nhttps://room-tba.uplbtools.me/?q=PSLH-A
END:VEVENT
```

---

## Implementation sketch

```ts
// packages/planner-core/src/calendar-export.ts

export type CalendarExportInput = {
  plan: Plan;
  blocks: GridBlock[];
  term: { startDate: string; endDate: string; label: string };
  roomTbaBaseUrl: string;
  conflicts?: Conflict[];
};

export function buildICS(input: CalendarExportInput): string;

export function buildGoogleCalendarTemplateUrl(block: GridBlock, term: Term): string;
```

**Dependencies:** none beyond `planner-core`: hand-roll ICS or use a tiny library (e.g. `ics` npm package) after bundle size check.

**UI (`apps/web`):**

- Toolbar button: **Export to Google Calendar** (triggers `.ics` download; label names Google explicitly).
- Subtext: "Also works with Apple Calendar and Outlook."
- Filename: sanitize `planName`, ASCII-only.

**Tests:**

- Golden-file ICS for one LEC + one LAB plan.
- RRULE `BYDAY` matches `Th` vs `T` parsing.
- Term `UNTIL` respects end date.
- Escaped commas/newlines in `DESCRIPTION`.
- Google template URL encodes `dates` as `YYYYMMDDTHHmmssZ` (UTC).

---

## Privacy

- Export runs **client-side only**: plan data never uploaded for calendar generation.
- No instructor names in `DESCRIPTION` (consistent with Room TBA PII policy).
- Exported file contains course codes and room codes the student already chose to plan.

---

## Acceptance criteria (Phase 2)

- [ ] Export button downloads valid `.ics` for a 5-course plan.
- [ ] Import into Google Calendar shows recurring events on correct weekdays/times (Asia/Manila).
- [ ] Events include room code and Room TBA link in description.
- [ ] Re-exporting the same plan produces stable `UID`s (Google dedupes updates on re-import where supported).
- [ ] Unit tests cover at least 5 schedule shapes including `MTWTHFS` intensive blocks.

---

## References (internal)

- Grid blocks: [IMPLEMENTATION_PLAN.md §4](./IMPLEMENTATION_PLAN.md)
- Term metadata: [ROOM_TBA_INTEGRATION.md §/api/terms](./ROOM_TBA_INTEGRATION.md)
- Slot parsing: Room TBA `src/lib/schedule-import/day-stops.ts` (port target)
