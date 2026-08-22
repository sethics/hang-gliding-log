# Hang Gliding Flight Log

Personal flight log for tracking USHPA H2 → H3 progress, plus per-glider
setup / preflight / takedown checklists.

Single file, no build step, no dependencies. Open `index.html` and it works.

## Running it

```bash
cd ~/hang-gliding-log
python3 -m http.server 8080
```

Then open `http://localhost:8080` — or on your phone, `http://<this-machine-ip>:8080`
while on the same wifi. Add to home screen for a full-screen app feel.

Opening the file directly (`file:///home/sethm/hang-gliding-log/index.html`)
also works. The only difference is the Google Fonts request, which needs a
network connection either way; without it the app falls back to system fonts
and still functions.

## Storage

Data lives under the key `hg-logbook-v1`. The `store` object at the top of the
script picks a backend at runtime:

- `window.storage` when running as a Claude artifact
- `localStorage` when running as a local file or off a local server

Same JSON shape either way, so an export from one restores into the other.

**localStorage is per-origin.** `file://` and `http://localhost:8080` are
different origins, and your phone's browser is a different store again. Pick
one way to open it and stay consistent, or use Export/Restore JSON to move
between them. Clearing browser data wipes the log.

**Do not commit your JSON exports to this repo.** It is public. An export
contains your USHPA number, emergency contact, and full flight history.
`.gitignore` blocks `*.json` and `*.csv` at the root so it can't happen by
accident.

Back exports up somewhere private instead — `~/Documents`, a secret gist, or
any private drive. This log backs a rating application; losing it means
reconstructing the reconstruction.

## Exporting

All five outputs are generated client-side; nothing is uploaded anywhere.

| Button | File | Contains |
|---|---|---|
| Flights CSV | `flight-log-YYYY-MM-DD.csv` | One row per flight, 11 columns. Clean tabular import for Sheets, Excel, or another logbook app. |
| Everything CSV | `logbook-full-YYYY-MM-DD.csv` | Sectioned: pilot card, H3 gate status, flights, witnessed tasks, every glider checklist step, rebuild checklist and notes. |
| Backup JSON | `logbook-backup-YYYY-MM-DD.json` | Complete state, pretty-printed. This is the one that restores. |
| Print / PDF | — | Paginated logbook for a rating official. Browser print dialog, Save as PDF. |
| Restore from JSON | — | Paste a previous JSON export to replace current state. |

Downloads use a Blob URL. If that's blocked (inside the Claude artifact
sandbox, for instance) the app falls back to showing the text with a Copy
button, so nothing is ever unreachable.

**Remember the exports are gitignored.** Save them somewhere private.

### The print view

Built on demand, rendered into `#printdoc`, styled entirely by the `@media
print` block. Repeating table headers via `display:table-header-group`, no
rows split across pages, and reconstructed entries marked with a double
dagger plus their cited source. Ends with a signature line for the observer.

Give this to whoever signs off the rating rather than a raw CSV. It reads as
a logbook and it is honest about which entries are estimates.

## Tabs

- **Log** — add / edit / delete flights, grouped by day. `Reconstructed` flags
  backfilled entries and requires naming the source.
- **H3** — USHPA membership card details, the three logged-requirement gauges,
  and the witnessed-task checklist.
- **Gliders** — per-glider setup, preflight, and takedown steps. Free text,
  one step per line, rendered as tickable checkboxes.
- **Rebuild** — evidence checklist for reconstructing past flights, plus notes.

## H3 logged requirements

Per USHPA's Pilot Proficiency Program (SOP 12-02):

| Gate | Required |
|---|---|
| Flying days | 30 |
| Flights | 90 |
| Solo airtime | 10 hours |

Plus witnessed tasks, the Intermediate written exam, and a signed waiver.
Tandem time does not count toward the solo hours — the app tracks it separately.

The task list in the app is a **paraphrase** for tracking convenience. The
current SOP from ushpa.org is what an observer signs against. Verify before
applying.

## Glider checklists

Ship empty deliberately. Do not fill these from memory or from a generic
online checklist — assembly steps are model-specific and getting them wrong
is not a software bug. Walk a setup with an instructor, write down what they
actually do, cross-check against the manufacturer's manual.

## Data shape

```
{
  version: 1,
  flights: [{ id, date, min, site, glider, launch, agl, witness,
              solo, est, src, notes }],
  gliders: [{ id, name, where, setup, pre, down }],
  tasks:   { <index>: bool },        // H3 witnessed tasks
  steps:   { "<gliderId>|s|p|d|<i>": bool },
  rebuild: { <key>: bool },
  rebuildNotes: "",
  pilot:   { name, num, rating, rdate, exp, skills,
             official, waiver, ice, notes }
}
```

`state.pilot` holds USHPA card details in plain text. Membership number and
rating are fine here. Don't put anything password-grade in it.

## Ideas not built yet

- Import a CSV instead of hand-entering the backfill
- Filter the log by site or glider
- Service-worker offline cache so it loads on the mountain with no signal
- H4 gates once H3 is done (250 flights, 80 days, 75 hours)
