# Meeting-Schedule-Helper

A self-contained, browser-based tool for planning recurring meeting series. It was primarily designed to aide in scheduling meetings within the higher education space i.e., faculty meetings, committees, debrief sessions, etc. across a term. The tool schedules based on a series of dates (i.e., academic calendar) and avoids organizational “no-go” dates and holidays, busy periods, and uses a cut-off date after which no further meetings are scheduled.

Any custom dates are uploaded as a CSV, so the tool works for any general time series once someone gives it the appropriate data.

---

## What it does

- Define up to **3 meeting series** at once (e.g., a reading circle, a debrief series, a committee)
- For each series, set a preferred **day pattern** (e.g., Mon/Wed, Tue/Thu), a **target number of meetings**, a **start date**, **time**, and **duration**
-	Optionally set a **fallback day**, used only if the primary pattern can't hit the target number of meetings before the cutoff.
- The tool auto-generates dates, skipping hard no-go dates entirely and flagging those that are soft-warnings (exam weeks or other custom events).
- Edit any generated date or time by hand. Edits are re-validated live against the calendar data you previously uploaded and will display a badge if you land on a hard warning or soft warning date. 
- **Review all series together** on one combined, color-coded timeline
- **Export** to CSV (spreadsheet-friendly) or ICS (imports into Outlook/Google/D2L calendars)
- Autosaves everything as you work, and reloads it next time you open the file

---

## Enforcement Rules

| Rule | Behavior |
|---|---|
| Organizational no-go dates (holidays, breaks, closures, etc.) | **Hard stop** — never scheduled |
| Any date after cut-off date| **Hard stop** — never scheduled |
| The cut-off date itself | Allowed, but flagged with a badge |
| Busy periods | **Soft warning** — flagged, still allowed |
| Manual edits onto any of the above | Allowed (it's your call), but clearly flagged so it doesn't slip through unnoticed |

---

## Using it

1. **Calendar Data tab** — upload a calendar CSV (or use the built-in Fall 2026 / Spring 2027 default to start). Download the current calendar first if you want to edit it rather than start from scratch.
2. **Build a Series tab** — add a meeting series, set its pattern/count/dates, click Generate.
3. **Review & Export tab** — see everything combined, export CSV/ICS.

### Calendar CSV format

One row per rule. Uploading a file replaces whatever calendar is currently loaded.

```
Semester,Type,StartDate,EndDate,Label
Fall 2026,SPAN,2026-08-17,2026-12-07,
Fall 2026,CUTOFF,2026-12-07,2026-12-07,
Fall 2026,NOGO,2026-09-07,2026-09-07,Labor Day
Fall 2026,NOGO,2026-11-25,2026-11-27,Thanksgiving Break
Fall 2026,SOFT,2026-10-02,2026-10-08,16-wk midterms
```

- **SPAN** — Start date of time period (first day of classes) - last date meetings could ever run (sets the timeline)
- **CUTOFF** — the hard no-meetings-after date
- **NOGO** — a hard blackout date or range (holiday, break, closure, conference, anything). `HOLIDAY` is also accepted as an alias.
- **SOFT** — a flagged-but-allowed window (needs a Label)

Dates can be `YYYY-MM-DD` or US `M/D/YYYY` — the parser normalizes either.

Add a new semester by adding rows with a new `Semester` name. Delete rows to retire an old one. Lines starting with `#` are treated as comments (even if a spreadsheet program wraps them in quotes on save).

---

## Tech stack

- **Single self-contained HTML file** — no build step, no server, no external JS framework
- **Vanilla JavaScript** for all logic (date generation, CSV parsing, CSV/ICS export)
- **Google Fonts** (Montserrat, JetBrains Mono) loaded via CDN link
- **Browser File API** (`FileReader`) for CSV upload; `Blob` + object URLs for CSV/ICS export/download
- **Browser `localStorage`** for autosave/reload — tied to the browser/device the file is opened in, not shared or synced between users or machines

No external JS dependencies, no npm, no package.json. The entire application is one `.html` file that runs client-side in the browser it's opened in.

### Rough structure inside the file

- **Calendar layer** — `parseCalendarCSV()` turns uploaded CSV text into a `SEMESTERS` object (span, cutoff, no-go dates, soft windows) keyed by semester name. `isHardBlackout()`, `hardFlagLabel()`, `cutoffFlagLabel()`, and `softFlagLabel()` answer "is this date okay, and why/why not."
- **Generation logic** — `generateSeriesDates()` walks day-by-day from a series' start date to the semester cutoff, first trying the primary day pattern alone, then folding in the fallback day only if needed to hit the target count.
- **State** — a simple in-memory `state` object holding all series (name, semester, pattern, generated dates, per-meeting overrides), persisted to `localStorage` after every change.
- **Rendering** — plain DOM string templating (`innerHTML` + event listener re-binding) for the three tabs: Build a Series, Review & Export, Calendar Data.
- **Export** — `exportCSV()` / `exportICS()` build the file contents directly from `state` and trigger a browser download.

---

## Known limitations

- Maximum of 3 concurrent meeting series
- Date format support is limited to ISO and US `M/D/YYYY` (not other locale date formats)
- Autosave (`localStorage`) is tied to the specific browser and device where the file is opened — it isn't shared between different people's copies, and moving the file to a new computer or browser starts with a blank slate (export CSV/ICS to carry data over)
- No timezone handling — all times are treated as local/wall-clock time

---

*Originally built for a specific higher education institution. However, the tool is generalized to accept any institution (or organizations) calendar data via CSV upload.*
