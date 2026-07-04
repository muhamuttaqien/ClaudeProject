# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project overview

A small collection of standalone, single-file HTML browser games. There is no build system, package manager, or test suite — each game is a single `.html` file with CSS and JavaScript inlined, meant to be opened directly in a browser or served statically.

Current games:
- `tic-tac-toe.html` — 3x3 grid, win/draw detection, scoreboard (X/O/draws) that persists across rounds via "New Round" / "Reset Scores" buttons.
- `connect-four.html` — 7x6 grid with gravity-based piece drops, animated falling pieces, a 2-player mode and a vs-Computer mode (minimax with alpha-beta pruning, 3 difficulty levels), win/draw detection, and a persistent scoreboard.

## Running a game

No install step is required. Either open the `.html` file directly in a browser, or serve the directory so relative paths/dev tools behave normally:

```bash
python3 -m http.server 8080
```

Then visit `http://localhost:8080/tic-tac-toe.html` or `http://localhost:8080/connect-four.html`.

`.claude/launch.json` defines this same static server (`static-server`, port 8080) for use with the preview tooling.

## Architecture notes (per game file)

Each game is fully self-contained (`<style>` and `<script>` inline in the one `.html` file) — there is no shared code between games. When editing one, all relevant logic is in that single file.

Common structure inside each game's `<script>`:
- A 2D/1D array holds board state; DOM is derived from it, not the other way around.
- A single `handleMove`-style entry point validates the move, mutates board state, then triggers rendering/animation.
- Win detection scans all lines (rows/cols/diagonals) for 4-in-a-row (Connect Four) or 3-in-a-row (Tic Tac Toe) from board state — it does not rely on DOM classes.
- A `scores` object tracks wins/draws across rounds; "New Round" resets board state only, "Reset Scores" resets both board and scores.

Connect Four specifics:
- Pieces animate by absolutely positioning a `.piece` div and transitioning its `top` from above the board down to its landing row (`animateDrop`), with duration scaled by landing row so lower slots take slightly longer (simulated gravity).
- **Input is locked (`boardWrapper.classList.add('locked')`) the instant a move is accepted, and only unlocked once the next player's turn actually begins** (in `afterMove`). This is load-bearing: because the turn switch is deferred until the drop animation's `setTimeout` resolves, removing the lock early (or locking too late) reopens a window where a fast double-click lets one player drop two pieces before the turn passes — this was a real bug found during development, not a hypothetical.
- The AI (`makeAIMove`) uses minimax with alpha-beta pruning (`minimax()`) over a cloned board (`cloneBoard`), scored via `scorePosition`/`scoreWindow` (center-column bias + weighted window scoring). Difficulty maps to search depth: easy = heuristic win/block/random only (no search), medium = depth 4, hard = depth 6. When adding difficulty levels or tuning strength, adjust depth in `makeAIMove`, not the heuristic weights, unless the evaluation itself needs to change.
