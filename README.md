PB Schedule Creator (index.html)
================================

Single-file, client-side pickleball schedule generator. Paste a list of players, set courts and rounds, and click Generate to produce per-round pairings. The generator prioritizes balanced court time, rotated sit-outs, and minimizing repeated partnerships/opponents.

What's new (recent)
-------------------

- Suggestions auto-compute when you change players, courts, rounds, or the "Require exact equality" toggle.
- "Require exact equality" (checkbox) enforces that every player gets the exact same number of games when mathematically possible.
- Suggestions banner shows minimal adjustments (rounds / courts) that would make exact equality possible; Apply updates the inputs. An Undo is available after applying.
- Pretty Export: download a visually formatted HTML export of the generated rounds & courts.
- Robustness improvements:
  - Deterministic fallback group selection when randomized selection struggles.
  - Defensive team filling so courts are always populated.
  - Best-effort backtracking pairing to avoid duplicate partnerships (with guarded fallbacks).
  - Post-generation assert-and-repair pass to fix duplicated players-per-round.
  - More retry attempts and console diagnostics for exact-scheduler failures.

Files
-----

- `index.html` — The single-page app (UI + scheduler + exports).
- `README.md` — This file.

Quick start
-----------

1. Open the app:
   - Double-click `index.html` or serve locally:

     ```bash
     python3 -m http.server 8000
     open http://localhost:8000/index.html
     ```

2. Controls:
   - Player names: paste newline- or comma-separated names.
   - Courts: number of courts (4 players per court).
   - Rounds: number of rounds.
   - Require exact equality: if checked, generator will only accept configurations that allow exact per-player game counts (or you can Apply a suggested small change).
   - Generate: click to build the schedule.
   - Export: download a plain-text schedule.
   - Pretty Export: download an HTML file that visually mirrors the on-page rounds & courts.

Behavior & implementation notes
--------------------------------

- Players-per-round = `numCourts * 4`. If fewer players than that are provided the UI warns and stops generation for that configuration.
- Exact scheduler: used when total play slots (numRounds *numCourts* 4) divides evenly by player count — the scheduler tries multiple randomized attempts (with deterministic fallback) to produce exact equality.
- Quota scheduler: used when exact equality isn't possible — distributes base plays to all players and gives one extra to a small set so the max difference is 1.
- Per-round uniqueness: no player is placed twice in the same round.
- Duplicate partnerships: the generator uses a bounded backtracking routine to avoid repeating partners. If the backtracker cannot find a pairing within its attempt budget, it falls back to randomized pairing for that round.
- Assertion & repair: after generation, a repair pass replaces duplicates (players appearing >1 in a round) with suitable substitutes and recomputes stats.
- Impossibility check: an early check compares required team pairings with the number of unique pairs available and will warn if the schedule is impossible without duplicate partnerships.

Pretty Export
-------------

- Click "Pretty Export" after generating to download an HTML file that visually represents rounds, courts, teams, and sitting-out players.
- The exported HTML is standalone and useable for printing or sharing.

Troubleshooting
---------------

- Generate does nothing / UI seems stuck: open DevTools → Console for errors. The Generate button is disabled while the generator runs to prevent concurrent execution.
- "Require exact equality" warning: if exact equality is impossible with current inputs, use the Suggestions banner to Apply a minimal change (or uncheck the toggle).
- If exact-scheduler repeatedly fails: the app logs per-attempt min/max player counts to the console for diagnosis. If failures persist, consider changing rounds/courts or use the Suggestions to find a nearby valid configuration.

Limitations & next steps
------------------------

- "Never duplicate partnership" is best-effort. Guaranteeing zero repeated partnerships requires a global exhaustive solver (exponential) or ILP; planned as an optional, higher-effort feature.
- Suggestion search window is modest (currently small deltas around current rounds/courts). If needed, increase the search window or try manual adjustments.
- No automated unit tests yet. Adding reproducible JS tests would help lock behavior.

Developer notes
---------------

Primary functions to inspect in `index.html`:

- `generateTournament()` — orchestrates scheduling and rendering.
- `scheduleWithExactPlays()` / `scheduleWithPlayQuotas()` — slot-based schedulers.
- `selectUniqueGroup()` — per-round unique player selection from remaining quotas.
- `makeTeamsNoDuplicate()` — backtracking pairing to avoid repeating partners.
- `assertAndRepairTournament()` — post-generation repair pass.
- `prettyExportTournament()` — builds and downloads the visual HTML export.

License
-------

Reuse and adapt freely. No explicit license file is provided.
