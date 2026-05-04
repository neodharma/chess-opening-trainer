# Chess Opening Trainer

A single-page browser app for drilling chess openings against Stockfish. You set up a starting line, the engine plays a strong response, and you have to find the best continuation. Everything runs locally — Stockfish ships as a WASM bundle and progress lives in `localStorage`.

Live: pushed to whatever Vercel project the repo is wired to (see `vercel.json`).

## How it works

1. **Set up an opening.** Type a line like `1. e4 e5 2. Nf3 Nc6` into the input, drag pieces on the board, or click one of the preset buttons (Italian, Sicilian, Ruy Lopez, etc.). The line should end on *your* last move — training starts with the engine's reply.
2. **Configure the session.**
   - **Moves per attempt** (1–15): how many of your moves the streak demands before the round resets.
   - **Eval tolerance** (0.05–0.50 pawns): how much eval drop from the best move is still accepted as correct.
   - **Engine depth** (10–22): Stockfish search depth for evaluating positions and picking moves.
   - **Engine move selection**: how the engine chooses among near-best replies.
     - *Top move only* — always plays Stockfish's #1 choice.
     - *Random (within 30cp)* — uniformly samples any move within 0.30 pawns of best.
     - *Weighted random* (default) — samples in proportion to closeness-to-best, so #1 is most likely but other reasonable moves still appear.
     - *Target weaknesses* — biases toward replies that lead into positions where you've previously played poorly (drawn from move-stat history).
3. **Hit Start.** The opening replays on the board, the engine responds, and you're on the clock.
4. **Make your move.** Drag or click-to-move. The trainer evaluates your move against Stockfish's options:
   - Within tolerance → ✅ counted as correct, streak continues, engine plays its next move.
   - Outside tolerance → ❌ shown the move you played vs. the moves that would have been accepted, with their evals. The attempt resets.
5. **Run streaks.** Stats are kept per opening line + per side: current streak, best streak, attempts, and per-move correctness rates (visible on `stats.html`).

## Features

- **In-browser Stockfish NNUE 16** — `stockfish-nnue-16.js` + `.wasm`. No server required.
- **Eval cache** keyed by position (move counters stripped) so revisiting the same line is instant. Capped at 2000 entries.
- **Move-level stats** logged per (opening, side, move number, played move) so the *Target weaknesses* mode and the stats page have data to work with.
- **Last-move highlight** on `from` and `to` squares. Selected piece and legal-move squares are also marked. All three highlights pull a per-theme accent (`--hl`) so they auto-contrast with whatever board palette is active.
- **Themes** — two independent dropdowns in the header:
  - *Board palette* (15 options): Greyscale, Brown, Green, Walnut, Slate, Blue, Marble, Olive, Purple, Ice, Charcoal, Sepia, Forest, Sand, Cherry. Each option previews as a 2×2 swatch.
  - *Piece set* (22 options, served from the lichess `lila` repo via jsDelivr): Cburnett, Merida, Maestro, Staunty, Alpha, California, Cardinal, Celtic, Chess7, Chessnut, Companion, Cooke, Dubrovny, Fantasy, Fresca, Gioco, Governor, Leipzig, Pirouetti, Spatial, Tatiana, plus **Blindfold** (transparent SVG — pieces are still draggable, you just can't see them).
- **Persistence** in `localStorage`:
  - `chess-trainer-stats` — per-opening streaks
  - `chess-trainer-movelog` — per-move correctness history
  - `chess-trainer-eval-cache` — Stockfish eval cache
  - `chess-trainer-settings` — slider values
  - `chess-trainer-board` / `chess-trainer-pieces` — theme selections

## Files

| File | Role |
|---|---|
| `index.html` | The whole trainer — UI, state, Stockfish glue, theming. |
| `stats.html` | Per-move correctness breakdown for a chosen opening + side. |
| `stockfish-nnue-16.js` / `.wasm` | Multi-threaded Stockfish (preferred). |
| `stockfish-nnue-16-single.js` / `.wasm` | Single-threaded fallback when `SharedArrayBuffer` / cross-origin isolation isn't available. |
| `vercel.json` | Sets COOP/COEP headers so the multi-threaded WASM build can use `SharedArrayBuffer`. |

## Stack

- [chessboard.js](https://chessboardjs.com/) for the board UI (CDN-loaded)
- [chess.js](https://github.com/jhlywa/chess.js) for move legality + state (CDN-loaded)
- Stockfish NNUE 16 WASM (bundled in the repo)
- jQuery for the DOM glue
- Maple Mono via `@fontsource/maple-mono` (CDN)
- Lichess piece SVGs via `cdn.jsdelivr.net/gh/lichess-org/lila` (CDN)

No build step. Open `index.html` in a browser, or serve the directory (`python3 -m http.server`) and load `localhost:<port>`.
