# tiny-chess-01 — v8 @ 60k nodes (Zobrist TT)

A small deterministic chess engine for the make.fun tiny-chess-01
bounty. 0x88 board, negamax + alpha-beta, iterative deepening,
quiescence search, MVV-LVA + killer-move ordering, and a transposition
table keyed by an incremental Zobrist hash. Stdlib only, deterministic.

## Why v8: the safe resubmit

The v5@120k entry scored 0 on the server — its practice grade was
rejected, most likely because the grading host was too slow and the
1,800 s wall limit expired. v8 halves the node budget to 60,000 per
move, so its wall time is 448 s shorter: a slow host must be 2.2x
slower to kill v8, versus 1.43x for v5. v8 is 138 Elo weaker than
v5@120k locally (1,326 vs 1,464) but far safer on time.

## Trial K re-run (2026-11-14)

Sealed scorer, practice trial-0, 8 games: 3 won, 1 drawn, 4 lost
(3.5/8), sealed Elo 1,326. Total wall 811 s, max move 2.42 s,
0 forfeits. Passes all three gates: Elo >= ~1,300, wall < 1,000 s,
max move < 5 s. Per-opponent: depth-2 W, depth-3 L, depth-4 W,
nodes-100k D, nodes-250k L, nodes-500k W, nodes-1M L, nodes-2M L.

## Development and validation

Trial K tested the Zobrist TT: +1 depth at ~1-3% speed cost. The
first attempt (v8@120k) was destroyed by machine overload, not the
TT — games lost on time as load spiked to 7.34 on 1.5 shared CPUs.
The re-run at 60k nodes in a quiet window (load 0.77-2.65) passed
all three gates. The TT's +1 depth did not pay for the halved node
budget in Elo (1,326 vs v5@120k 1,464), but the time safety margin
is what this resubmit is for.

## How it works

- 0x88 board, signed ints; move = from | (to<<7) | (promo<<14).
- Negamax + alpha-beta, iterative deepening, cumulative 60,000-node
  budget per move (depth 1 unlimited; an over-budget iteration is
  dropped).
- Quiescence: captures and promotions; all evasions when in check.
- Ordering: MVV-LVA captures, promotions, killer moves, TT move.
- TT: dict keyed by the Zobrist hash, storing depth, score, flag and
  best move; hash updated incrementally on make/unmake.
- Eval: material P100 N320 B330 R500 Q900 + piece-square tables
  (house formulas) + bishop pair +30 + king phase interpolation.
- Env protocol: POST /game, /move, /resign at MAKEFUN_ENV_URL.

## Tests

- Perft: all 6 standard positions, depths 1-3, exact.
- Incremental-hash differential vs from-scratch hash: no drift,
  including after unmake.
- Mate-in-1 (Scholar's mate h5f7) found; stalemate: 0 legal moves.
- Deterministic: repeated runs give the same move and node count.
- Eval and best move identical to v5 on 4 test positions.

## Packing

The entrypoint `main.py` is produced by the kit's `makefun compress`
from the readable source `main_min10_60k.py` (7,886 bytes). The source
is sent beside the transcripts, not committed to the repo.
