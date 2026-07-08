# Meeting-Schedule-Helper

# Meeting Schedule Helper

A self-contained, browser-based tool for planning recurring faculty meeting series (reading circles, committees, debrief sessions, etc.) across a term, automatically respecting the real academic calendar — institutional no-go dates, exam weeks, and the grades-due cutoff.

Built for Faculty Development / eCampus use, but designed to be reusable by anyone: the academic calendar itself is uploaded as data, not hardcoded, so the tool works for any semester once someone points it at the right CSV.

---

## What it does

- Define up to **3 meeting series** at once (e.g., a reading circle, a debrief series, a committee)
- For each series, set a **day pattern** (e.g., Mon/Wed, Tue/Thu), a **target number of meetings**, a **start date**, **time**, and **duration**
- Optionally set a **fallback day** — used only if the primary pattern can't hit the target count before the cutoff
- The tool **auto-generates dates**, skipping hard no-go dates entirely and flagging (but not blocking) exam weeks
- **Edit any generated date or time by hand** — edits are re-validated live against the calendar (you'll see a badge if you land on a holiday, past the cutoff, or on the grades-due day itself)
- **Review all series together** on one combined, color-coded timeline
- **Export** to CSV (spreadsheet-friendly) or ICS (imports into Outlook/Google/D2L calendars)
- **Autosaves** everything as you work, and reloads it next time you open the file

---

## The rules it enforces

| Rule | Behavior |
|---|---|
| Institutional no-go dates (holidays, breaks, closures, etc.) | **Hard stop** — never scheduled |
| Any date after grades are due | **Hard stop** — never scheduled |
| The grades-due date itself | Allowed, but flagged with a "Grades due this day" badge |
| Midterm / final exam weeks | **Soft warning** — flagged, still allowed |
| Manual edits onto any of the above | Allowed (it's your call), but clearly flagged so it doesn't slip through unnoticed |

---

## Using it

1. **Calendar Data tab** — upload a calendar CSV (or use the built-in Fall 2026 / Spring 2027 default to start). Download the current calendar first if you want to edit it rather than start from scratch.
2. **Build a Series tab** — add a series, set its pattern/count/dates, click Generate.
3. **Review & Export tab** — see everything combined, export CSV/ICS.

### Calendar CSV format

One row per rule. Uploading a file **replaces** whatever calendar is currently loaded.

```
Semester,Type,StartDate,EndDate,Label
Fall 2026,SPAN,2026-08-17,2026-12-07,
Fall 2026,CUTOFF,2026-12-07,2026-12-07,
Fall 2026,NOGO,2026-09-07,2026-09-07,Labor Day
Fall 2026,NOGO,2026-11-25,2026-11-27,Thanksgiving Break
Fall 2026,SOFT,2026-10-02,2026-10-08,16-wk midterms
```

- **SPAN** — first day of classes → last date meetings could ever run (sets the timeline)
- **CUTOFF** — the hard no-meetings-after date
- **NOGO** — a hard blackout date or range (holiday, break, closure, conference, anything). `HOLIDAY` is also accepted as an alias for older files.
- **SOFT** — a flagged-but-allowed window (needs a Label)

Dates can be `YYYY-MM-DD` or US `M/D/YYYY` — the parser normalizes either (this matters because Excel silently reformats ISO dates to `M/D/YYYY` when you edit and re-save a CSV).

Add a new semester by adding rows with a new `Semester` name; delete rows to retire an old one. Lines starting with `#` are treated as comments (even if a spreadsheet program wraps them in quotes on save).

---

## Tech stack

- **Single self-contained HTML file** — no build step, no server, no external JS framework
- **Vanilla JavaScript** for all logic (date generation, CSV parsing, CSV/ICS export)
- **Google Fonts** (Montserrat, JetBrains Mono) loaded via CDN link
- **Browser File API** (`FileReader`) for CSV upload; `Blob` + object URLs for CSV/ICS export/download
- **Artifact storage API** (`window.storage`) for autosave/reload — personal-scope only, not shared between users

No external JS dependencies, no npm, no package.json. The entire application is one `.html` file that runs client-side in the browser it's opened in.

### Rough structure inside the file

- **Calendar layer** — `parseCalendarCSV()` turns uploaded CSV text into a `SEMESTERS` object (span, cutoff, no-go dates, soft windows) keyed by semester name. `isHardBlackout()`, `hardFlagLabel()`, `cutoffFlagLabel()`, and `softFlagLabel()` answer "is this date okay, and why/why not."
- **Generation logic** — `generateSeriesDates()` walks day-by-day from a series' start date to the semester cutoff, first trying the primary day pattern alone, then folding in the fallback day only if needed to hit the target count.
- **State** — a simple in-memory `state` object holding all series (name, semester, pattern, generated dates, per-meeting overrides), persisted to `window.storage` after every change.
- **Rendering** — plain DOM string templating (`innerHTML` + event listener re-binding) for the three tabs: Build a Series, Review & Export, Calendar Data.
- **Export** — `exportCSV()` / `exportICS()` build the file contents directly from `state` and trigger a browser download.

---

## Known limitations

- Maximum of 3 concurrent meeting series
- Date format support is limited to ISO and US `M/D/YYYY` (not other locale date formats)
- `window.storage` autosave is personal to whoever has the file open — it isn't shared between different people's copies
- No timezone handling — all times are treated as local/wall-clock time

---

*Originally built for eCampus Faculty Development meeting planning; generalized to accept any institution's calendar data via CSV upload.*
