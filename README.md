
PB Schedule Creator (index.html)
================================

This is a single-file, client-side pickleball schedule generator. Give it a list of players, a number of courts and rounds, and it will produce per-round pairings while trying to balance court time, rotate sit-outs, and minimize repeated partnerships and opponent repeats.

What’s new (recent changes)

---------------------------

- Require exact equality toggle: a UI checkbox ("Require exact equality") lets you insist that every player plays exactly the same number of games. When enabled the generator will either produce an exact-equality schedule or inform you it's impossible with the current configuration.
- Suggestions panel: a small helper computes minimal changes to rounds/courts (within a small search window) to make exact equality possible, and you can apply those changes with one click. The app can auto-apply this suggestion at load when the toggle is enabled.
- Exact/quota schedulers: the generator now contains two slot-based schedulers:
  - An exact scheduler that assigns every player the same number of play slots when total slots divide evenly by number of players.
  - A quota scheduler that distributes slots so counts differ by at most 1 when exact equality isn't possible.
- Per-round uniqueness: selection of players for each round guarantees a player won't appear twice in the same round.
- Duplicate-partnership avoidance: when forming teams inside a round the generator uses a best-effort backtracking algorithm to avoid reusing partnerships from earlier rounds. If the backtracker cannot find a pairing quickly it falls back to randomized pairing.
- Assertion & repair pass: after generation a repair pass fixes any remaining duplicated player occurrences inside a round (preferentially replacing with low-played players) and recomputes statistics.
- Robustness improvements (recent): the schedulers are now more defensive to reduce failures when the randomized heuristics struggle. Changes include:
  - Deterministic fallback selection: if the randomized unique-group selector can't find a group within its attempt budget the code now falls back to a deterministic greedy pick so generation can continue when a solution likely exists.
  - Defensive team filling: team arrays are always initialized and shortfalls are filled with randomized pairs so courts are fully populated (avoids malformed rounds).
  - Partnership guards: partnership updates are guarded against malformed teams to avoid runtime errors.
  - More attempts & smarter fallback: the exact scheduler retries many more randomized attempts (increased attempt cap) and, when exact-equality cannot be achieved by the heuristics, the generator will automatically fall back to the quota scheduler if "Require exact equality" is not checked.
  - Console diagnostics: when exact scheduling attempts fail the app logs per-attempt min/max produced counts to the browser console to help debugging.

Files
-----

- `index.html` — The single-page app. Open it in a browser to use the UI.

Quick start (open and run)
-------------------------

1. From Finder / double-click: open `index.html` in your browser.

2. Or serve locally for a stable experience (recommended):

```bash
# from the project root
python3 -m http.server 8000
# then open in your browser (macOS)
open http://localhost:8000/index.html
```

How to use the UI
-----------------

1. Paste or type player names into the "Player names" textarea. Names can be separated by newlines or commas.
2. Set the number of courts (each court uses 4 players: two teams of two).
3. Set the number of rounds.
4. Toggle "Require exact equality" if you want every player to have exactly the same number of games. If the current configuration cannot satisfy that constraint you can use the Suggestions panel to compute a minimal change (small adjustments to rounds/courts) and apply it.
5. Click "Generate" to build the schedule. A small overlay appears while the generator runs.
6. Click "Export" to download a plain-text schedule file.

Behavior and implementation notes
---------------------------------

- Players-per-round = `numCourts * 4`. If you provide fewer players than that the UI will alert and stop generation. If you provide more players, the extra players will sit out that round; the generator attempts to rotate sit-outs fairly.
- Exact scheduler: when total play slots (numRounds *numCourts* 4) divides evenly by the number of players the generator will try to give each player exactly the same number of games (it will retry a few randomized attempts before failing).
- Quota scheduler: when exact equality is impossible the generator assigns a base number of games to every player and gives one extra game to a small set of players so the difference between any two players' game counts is at most 1.
- Duplicate partnerships: the code builds and maintains partnership maps and tries to avoid repeating partners. The backtracking pairing routine is best-effort and bounded by attempt limits; in some pathological schedules it may fall back to randomized pairings. There's also an early impossibility check that alerts when the total number of team pairings would exceed the number of unique player pairs.
- Repair pass: after generation a lightweight repair pass fixes any player who appears more than once in the same round by swapping in a suitable replacement (prefer low-played players). If the repair cannot resolve all issues you are warned in the UI/console.

Limitations & next steps
------------------------

- The "never duplicate partnership" goal is implemented as a best-effort backtracker with attempt caps. Guaranteeing absolute avoidance would require a global exhaustive solver (exponential search) or an integer-program (ILP) formulation and solver.
- The suggestion search currently looks in a small window around the current rounds/courts values (±6 rounds, ±3 courts). If you have special constraints you may need to adjust those values directly.
- There are no automated unit tests in this repo yet. Adding reproducible JS tests would help verify no-duplicate and exact-equality properties across seeds.

Troubleshooting
---------------

- "Not enough players" alert: increase the player list or reduce the number of courts.
- "Require exact equality" warning: if the toggle is checked and exact equality is impossible the UI will either suggest a minimal change or prompt you to adjust rounds/courts.
- If the generator fails to produce a round or seems to hang, check the browser console for warnings/errors. The app uses randomized attempts and may log retries.

Development notes
-----------------

- The main application lives entirely in `index.html`. Key functions to look at when changing behavior are:
  - `generateTournament()` — top-level orchestration.
  - `scheduleWithExactPlays()` and `scheduleWithPlayQuotas()` — slot-based schedulers.
  - `selectUniqueGroup()` — picks a per-round unique set of players from remaining quotas.
  - `makeTeamsNoDuplicate()` — backtracking routine to try to avoid repeating partnerships.
  - `assertAndRepairTournament()` — post-generation repair pass.
- If you want to implement a guaranteed solver for duplicate-free pairings, consider modeling the problem as an ILP and using a small WASM or remote solver, or implement a global backtracking search that assigns pairings across rounds (this is more complex but can provide guarantees).

License
-------

Feel free to reuse and adapt. No explicit license file is included in the repo.
