# tiny-chess-01 candidate #6

Bitboard engine, `main.py` 3,967 bytes (cap 4,096). Built on candidate #5
(3.0/8, Elo 1,253, tied with #2); target: BEAT submitted #2 (3.0/8, Elo 1,253).

## Changes vs #5

1. **PVS (principal variation search).** At the root the first move is
   searched full-window, the rest with a null window `(-bs-1, -bs)` and
   re-searched full-window only on fail-high. In `sr()` the same: first
   move full window, the rest null-window `(-a-1, -a)`, re-search on
   `s > a`. #5 searched every root move with a full window, which is why
   it only completed depth 4 in middlegame positions.
2. **LMR uses the null window.** The reduced-depth probe is now
   `(-a-1, -a)` instead of the full window #5 used; re-search at full
   depth/full window only if it beats alpha.
3. **Depth cap 8 -> 12** (`range(1, 13)`); the node budget decides.
4. **Node budget: 100,000 (unchanged from #5).** The first #6 trial used
   150,000 and LOST ON TIME in game 4 (one move 11,877 ms > 10,000 ms
   limit under trial load) — see below. 100,000 keeps the slowest move
   at 3,177 ms.

Kept from #5: history-aware repetition steering, null-move margin test,
check extension, killers, endgame king table, fixed node budget (no wall
clock — replay stays deterministic), packer NUL-as-`\x00` fix, driver
`saves q = g["game"]` before the /move retry loop.

## Trials (official practice trial-0, kit route)

- trial-c6 (BG=150,000): 2W-1D-5L = 2.5/8, Elo 1,173. LOSE. G4 lost on
  time at ply 9 (11,877 ms move); G2 win -> draw. Do NOT use.
- trial-c6b (BG=100,000): **5W-1D-2L = 5.5/8, Elo 1,598. BEAT #2.**
  G1 W vs depth-2 (41), G2 W vs depth-3 (112), G3 W vs depth-4 (87),
  G4 W vs nodes-100k (98, checkmate, up 13 material), G5 D vs
  nodes-250k (61, threefold), G6 L vs nodes-500k (89), G7 L vs
  nodes-1M (64), G8 W vs nodes-2M (200, adjudicated, rook vs bare king).
  Max think 3,177 ms, no time losses. Scores in `trial-c6b/scores/`.

## Tests

perft vs python-chess ALL OK; incremental-eval ALL PASSED; repetition
test PASS (avoids h7d3 at ply 61, plays h7e4).
