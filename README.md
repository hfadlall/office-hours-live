# 70-110 Business Computing — live office hours page

One file, `index.html`. It draws the weekly office-hours grid from a schedule
written into the file, then fetches a Google Sheet published as CSV and paints
this week's cancellations, moves and extra sessions on top of it.

Nothing to build, nothing to install. Open the file, or serve it anywhere.

---

## (a) For course assistants — adding a change from your phone

1. Open the **Office Hours Changes** sheet (bookmark it; the link is in our group chat).
2. Tap the first empty row and fill in `date` as `2026-09-01` — year-month-day, nothing else.
3. Put your name in `person` exactly as it appears on the schedule, and one of
   `cancelled`, `moved` or `extra` in `status`.
4. Put the session's usual start time in `time` (e.g. `15:30`). If you are moving it,
   put the new time in `new_time` (e.g. `16:00-17:00`); for an extra session, `time` and
   `new_time` are the start and end. Add a short `note` if students should know why.
5. That is it — the page picks it up within about five minutes. Do not delete old rows.

**If you have two sessions on the same day** (Oselumese on Thursdays), the `time`
column is what tells the page which one you mean. Leave it blank and the page will
not guess: it shows students "check with the CA" instead.

---

## (b) The sheet (Hussein)

The sheet is created, published as CSV, and already wired into the page — the link
sits on **line 425 of `index.html`** and was checked end to end: it answers `200` with
`text/csv` and `Access-Control-Allow-Origin: *`, so any browser on any host can read it.
Right now it holds only the header row, and the page correctly says
*"No changes this week ✓"*.

**The one step left:** open the sheet ▸ **Share** ▸ add each CA as **Editor**. Publishing
to the web only makes it *readable* by the page; it does not let anyone write to it.

If you ever re-publish and the link changes, replace the string on line 425:

```js
SHEET_CSV_URL: "https://docs.google.com/spreadsheets/d/e/…/pub?output=csv",
```

Keep the columns spelled as in `exceptions_template.csv` —
`date, person, status, time, new_time, note`. The page also accepts a few obvious
synonyms (`name` for `person`, `reason` for `note`) and can read the columns
positionally if the header row is ever deleted by accident.

> **Google caches the published CSV for about five minutes.** An edit is not instant —
> tell the CAs to add the row a little before it matters, not thirty seconds before.

If the link is wrong, missing, or Google is down, the page still draws the full base
schedule and says so quietly: *"Live updates unavailable — schedule shown is the
default."* It never shows a blank or broken page.

---

## (c) Changing the base schedule later

Open `index.html` and edit `BASE_SCHEDULE`, which starts at **line 455**. One line per
session:

```js
{ day: 0, person: "Darya", start: "10:00", end: "11:30" },
```

* `day` — `0` Sunday, `1` Monday, `2` Tuesday, `3` Wednesday, `4` Thursday.
* `start` / `end` — 24-hour, `"HH:MM"`.
* `person` — must match a name in `ROSTER` just above it (line 442).

To add or remove a **person**, edit `ROSTER`:

```js
{ name: "Sara", role: "ca", where: "ARC" },
```

`role` picks the colour and is one of `ca` (maroon), `ta` (navy), `prof` (green);
`where` is the room printed on their blocks and used for any extra session they add.

Save, reload, and press **T** — a hidden self-test runs and should say *26/26 passed*.
It checks, among other things, that every line of the schedule is internally
consistent. The legend under the title is plain HTML in the same file; edit it by hand
if the roles ever change.

Sessions that overlap are placed side by side automatically, so you do not need to do
anything special for Monday and Wednesday afternoons.

---

## What the page does with each row of the sheet

| status | what students see |
|---|---|
| `cancelled` | the block fades to 40%, is struck through, and says "Cancelled" plus your note |
| `moved` | the original block is struck through and says "Moved to …" plus your note |
| `extra` | a new block appears at `time`–`new_time` in that person's usual room, marked "Added" |

Only rows dated inside the current week (Sunday–Thursday, Asia/Qatar) change the grid.
Rows for a later week are collected into a collapsed *"n changes in a later week"* line
under the banner. Rows in the past are ignored.

Rows the page cannot use are skipped, never guessed at, and the reason is written to the
browser console (right-click ▸ Inspect ▸ Console) — a malformed date, a name that is not
on the schedule, a status that is not one of the three, a Friday or Saturday date, or a
row that could mean either of two sessions on the same day. One bad row never stops the
others from being applied, and never breaks the page.

The page is forgiving about small things: `canceled` and `cancelled` both work, `3:30`
finds a 15:30 session, `Abdul` is rejected as ambiguous (it could be Abdullah or Abdul
Hassib) but `Abdul Hassib` works, and quoted notes containing commas are fine.

---

## Odds and ends

* **Printing** — Cmd/Ctrl-P gives one A4 landscape page. This week's changes print with
  it; the "later weeks" list and the timestamp do not.
* **Phones** — the grid keeps all five days on screen and shortens the day names.
* **A thin line across today's column** marks the current time in Doha. Turn it off with
  `SHOW_NOW_LINE: false` in CONFIG.
* **The page re-checks the sheet** when you come back to a tab you left open for more
  than two minutes.
* **Press T** on the page at any time to run the built-in tests. Students will never
  find this by accident, and it is not linked from anywhere.

---

## Deploying

The repository is initialised and committed. GitHub CLI 2.98 is installed but not
logged in, so the push has not happened yet. In a **new** terminal (the installer put
`gh` on your PATH, which existing terminals will not have picked up), from this folder:

```powershell
gh auth login
gh repo create office-hours-live --public --source . --push
gh api -X POST "repos/$(gh api user -q .login)/office-hours-live/pages" -f "source[branch]=main" -f "source[path]=/"
```

That is it — the site appears at
`https://<your-github-username>.github.io/office-hours-live/` a minute or two later.
To check it is up:

```powershell
(Invoke-WebRequest "https://$(gh api user -q .login).github.io/office-hours-live/" -UseBasicParsing).StatusCode
```

Any static host works just as well; the page is one file with no dependencies.
