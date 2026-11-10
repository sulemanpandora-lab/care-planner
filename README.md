# Care Sprint Capacity Planner

**Pandora Digital · Care / SERVICEF**

A sprint capacity planning tool for the Care team: everyone marks their own leave, and the planner works out the team's real capacity and story points per sprint.

🔗 **Live:** https://sulemanpandora-lab.github.io/care-planner/
📘 **How to use it:** see the Confluence page, *Care — Sprint Capacity Planner*, under **AI Tools** in the Online space

> ⚠️ **Internal tool.** Built for the Care team at Pandora Digital. Please ask before adapting it for another team.

---

## Not a fork of the PVA planner

The two planners look alike and solve the same problem, but they are **separate codebases with different data shapes**. Do not copy code between them without translating it.

| | care-planner | pva-planner |
| --- | --- | --- |
| Sprint fields | `label`, `start`, `end` | `l`, `s`, `e` |
| Leave data | `data[personId][sprintId][date]` | `data[sprintId][memberId][date]` |
| `localStorage` | `carePlannerData` | `pva26c` |
| Excluded from capacity | `capacity:false` on the member | omitted from `SP_ROLES` |
| Rendering | DOM `createElement` | `innerHTML` strings |
| Saving | **automatic** | manual Save button |
| SP settings in the UI | no — fixed at 1 SP/day × 70% | yes |

They also have **separate Google Sheets and separate Apps Script projects**, so the two teams cannot collide with each other.

---

## Features

### Sprint navigation
- 2026 (25 sprints) and 2027 (26 sprints), all 2-week cycles, with a year toggle
- The current sprint is highlighted automatically

### Capacity calendar
- Every team member against every day of the sprint
- Click a working day to mark it as leave, click again to clear
- Weekends are excluded automatically
- Per-person capacity bar: green ≥80%, amber 60–80%, red <60%

### Story points
- Estimated SP per sprint, per group and per person
- Formula: `available days × FTE × 1 SP per day × 70%` — both figures fixed

### Velocity tracking
- **Delivered in S*XX*** — actual SP completed, entered by hand from Jira's Velocity Report
- **Avg velocity** — running average of everything entered

The "delivered" card always refers to the **previous** sprint, never the one on screen: the sprint being planned has not finished yet.

> Worth knowing: in November 2026 the estimate read 56 against an average velocity of 38.5, which looked like a broken formula. It was a stale roster. With the six people actually on the team the estimate came out at 37 against 38.5 actual. **Check the roster before suspecting the formula.**

### FTE
Per-person FTE, editable from a badge next to the name. Stored in `fteOverrides` **only when it differs from the roster**, so `BASE_GROUPS` stays the source of truth — picking the roster value again deletes the override.

### Roster vs. capacity
A member can carry `capacity:false`: still listed, but left out of the capacity maths. Used for long absences. Keeping them on the roster preserves their history and makes bringing them back a one-line change.

### Other
- **Plan longer leave** — one date range fills every working day across however many sprints it spans, including across the turn of the year
- Guests for borrowed team members
- Quarter view — every sprint in the year with capacity and estimated SP
- Share link with the current data encoded as a snapshot
- Dark mode follows the system setting

---

## Saving

**Care saves automatically**, about two seconds after any change. The Save button only forces it sooner.

Data lives in a shared Google Sheet, reached through a Google Apps Script web app, stored as one JSON blob chunked at 40,000 characters across rows (a single cell caps at 50,000). `localStorage` is a per-browser cache, not the source of truth.

### Three things protect the data

| Guard | What it stops |
| --- | --- |
| `LockService` | Two saves interleaving and leaving broken JSON in the sheet |
| Revision check | A save silently overwriting someone else's edit — it is refused instead |
| Empty-sheet rule | A client that never loaded replacing a populated sheet |

The last one matters most here. Because Care autosaves, no deliberate action stands between a failed load and a write: a teammate on a bad connection could open the app, have the fetch fail, click once, and replace everything. The app also shows a red **"Not connected to the team sheet"** bar and refuses to save while it is up.

**Recovery:** Google Sheets keeps full version history. File → Version history → See version history, restore, then press **⟳ Load** in the planner.

---

## Tech stack

| Layer | Technology |
| --- | --- |
| Frontend | Plain HTML + CSS + JS, one `index.html` |
| Hosting | GitHub Pages, auto-deploys from `main` |
| Storage | Google Sheets — JSON chunked down column A, tab `state` |
| API | Google Apps Script web app |

---

## Team roster

Deliberately not listed here — this repository is public. The current roster lives in `BASE_GROUPS` in `index.html` and, in readable form, on the Confluence page.

**Leave data must never be committed to this repository.** It belongs in the private Google Sheet the app syncs with.

---

## Making changes

1. Edit `index.html` and push to `main`. GitHub Pages redeploys in 1–5 minutes.
2. Run the regression suite first — it covers both planners.

### If sync stops working

1. Open the Apps Script project from the Google Sheet: **Extensions → Apps Script**
2. **Deploy → Manage deployments →** pencil icon
3. Check **Who has access**
4. Changing the code is not enough — you must deploy a **New version**, or `/exec` keeps serving the old one

### Dates: one trap worth knowing

`new Date('2026-10-29')` parses as **UTC**; `new Date('2026-10-29T00:00:00')` parses as **local**. Mixing a local parse with `toISOString()` shifts everything a day earlier in CET — which is exactly the bug that once made every leave day land one column too far left. Use the local form, and build date strings from `getFullYear()`/`getMonth()`/`getDate()`, never `toISOString()`.

---

*Maintained by Suleman Malik, Pandora Digital*
