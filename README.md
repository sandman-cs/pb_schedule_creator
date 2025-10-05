PB Schedule Builder
===================

A simple client-side HTML page that generates round-robin-style pickleball schedules. The generator accepts a list of player names, number of courts, and number of rounds and attempts to minimize repeated partnerships and opponent repeats while balancing court time and sit-outs.

Files
-----

- `PB Schedule Builder.html` — The main HTML file. Open it in a browser to use the UI.

Usage
-----

1. Open `PB Schedule Builder.html` in your browser. You can either:

   - Double-click the file in Finder (this opens it directly in your default browser), or

   - Serve the project directory and open the page in your browser (recommended for consistent behavior):

```bash
# from the project root
python3 -m http.server 8000
# then open in your browser (macOS)
open http://localhost:8000/PB%20Schedule%20Builder.html
```

2. In the page UI:
   - Paste or type player names into the "Player names" textarea. Names can be separated by newlines or commas.
   - Set the number of courts (each court uses four players: two teams of two).
   - Set the number of rounds.
   - Click "Generate" to build the schedule.
   - Click "Export" to download a plain-text schedule file.

Behavior and notes
------------------
- Players per round = `numCourts * 4`. If you provide fewer players than that, the page will alert and won't generate a schedule. If you provide more players, the difference are the players who will sit out each round (the script tries to rotate sit-outs fairly).
- Names are treated literally and not auto-deduplicated; use unique names for accurate stats.
- The generator attempts to minimize duplicate partnerships and repeated opponents but is heuristic-based — perfect optimality isn't guaranteed for all combinations.
- The default player list is prefilled in the UI from the `defaultPlayers` array inside `PB Schedule Builder.html`.

Troubleshooting
---------------
- "Not enough players" alert: increase the player list or reduce the number of courts.
- If the UI appears blank or controls do not work, open the browser console (Developer Tools) and check for script errors.
- If the schedule looks odd, try re-generating (the algorithm uses randomized shuffles to explore options).

Development / Next steps
------------------------
- Add automatic duplicate-name detection and normalization (trim/unique).
- Add save/load configuration (JSON) and printing-friendly view.
- Allow custom sit-out counts or custom court sizes.

License
-------
Feel free to reuse and adapt. No license file provided.

If you'd like, I can also:
- Add auto-cleanup for duplicate names,
- Add a print-friendly view/button,
- Or add a small JS unit test harness to validate the scheduler for specific player/court/round combinations.

