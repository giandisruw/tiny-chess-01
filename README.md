# tiny-chess-01 — 4 KB Python chess engine

A strong chess engine in a single `main.py` (3,856 bytes including a
one-line header comment, limit 4,096).
Entry for the make.fun bounty "Build the strongest chess engine in 4 KB of Python".

## The entry (v5 @ 120k nodes)

- Eval-optimized alpha-beta engine; trial-0 gauntlet **8.5/16, ~1,433 Elo**
  (Trial H, 2026-11-12) — above the ~1,360 house baseline.
- Same play as v2 (identical eval values and best moves on test positions),
  but ~45% faster, so more headroom under the 10 s/move cap on the
  grading machine.

## How it works

- **Board:** 0x88 (128 squares), signed ints (white +type, black -type).
- **Search:** negamax + alpha-beta, iterative deepening to depth 12 with a
  cumulative 120,000-node budget per move (depth 1 unlimited; an iteration
  that runs past the budget is dropped whole).
- **Quiescence:** captures and promotions; all evasions when in check.
- **Move ordering:** MVV-LVA captures, promotions, killer moves.
- **Eval:** material (P100 N320 B330 R500 Q900) + piece-square tables
  (computed from the house formulas) + bishop pair (+30) + king table
  interpolated by game phase.
- **Protocol:** talks to the env at `MAKEFUN_ENV_URL` (`POST /game`,
  `POST /move`, `POST /resign`). Deterministic: fixed node budget, no clock,
  no randomness, no model calls. Stdlib only.
- **Packing:** `main.py` is an lzma + base85 payload with a one-line
  unpacker (`import lzma,base64;exec(...)`) plus a header comment
  identifying the version. The readable source is `main_min7_120k.py`
  (7,208 bytes); see NOTES.md for the build.

## Tests (all on this machine, 2026-11-12/13)

- Perft: all 6 standard positions, depths 1-3, exact counts (start
  20/400/8902, Kiwipete 48/2039/97862, etc.). Move generation correct.
- Search: mate-in-1 (Scholar's mate h5f7) found; stalemate = 0 legal moves.
- Speed: ~49,600 nps (start), ~38,200 nps (middlegame). 120k nodes ~2.9 s/move.
- Practice trials vs the local env server (trial-0 gauntlet): see NOTES.md.
  Best result: v5 @ 120k, Trial H, 8.5/16 (~1,433 Elo).

## Development and validation

- v5 is an eval-optimized rewrite of v2 (same eval values and best moves
  on test positions), built to run the same search faster: 120,000 nodes
  in ~2.85 s/move vs ~4 s for v2 (~45% faster).
- Trial G first ran v5 at a 160k node budget and dropped to 5.5/16
  (~1,299 Elo); Trial H isolated the cause by re-running the same engine
  at 120k nodes on the same trial-0 gauntlet: **7 won, 3 drawn, 6 lost
  (8.5/16, ~1,433 Elo)** — exactly tied with v2, with identical
  per-opponent splits. The 160k budget, not the eval, was the problem.
- Validation: perft on 6 standard positions (depths 1-3, all exact),
  mate-in-1 found, stalemate = 0 legal moves, deterministic (Trial H
  games 1-9 reproduced exactly across two runs).
- Readable source: `main_min7_120k.py`; `main.py` is the packed build
  (3,856 bytes, limit 4,096). Full trial history in NOTES.md.

## License

MIT. See LICENSE.
