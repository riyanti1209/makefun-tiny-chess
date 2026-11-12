# Tiny Chess Engine — make.fun entry

Bounty: **Tiny chess 01** (`tiny-chess-01`) — $500 in USDC on Solana.
Closes **Tue 2026-11-25 00:00 WIB**. Account: **@riyanti1209**.

## The task (exact spec)

Write a chess engine in a single Python 3.13 file, `main.py`, of at most
**4,096 bytes**, that plays 8 games through the make.fun env
(`MAKEFUN_ENV_URL`, `/game` and `/move`), one against each of a fixed
gauntlet of eight house engines, and is rated by an Elo estimate.
Wall time 1,800 s per trial (~3 s/move on average, one move may take 10 s).

## This entry

`main.py` is **4,068 bytes**: an lzma + base85 self-extracting wrapper
(`import lzma,base64;exec(lzma.decompress(base64.b85decode('...')))`)
around a compact 0x88 chess engine (`engine_s.py`, 8,492 bytes).

The engine keeps: 0x88 board, alpha-beta + transposition table (keyed by
`tuple(board)`), quiescence search, check extension, MVV-LVA + TT-move
ordering, piece-square tables, bishop-pair bonus, castling, en passant,
repetition + insufficient-material draws, iterative deepening, and an
80,000-node cap per move. The full env protocol (`/game`, `/move`, same
stderr lines) is intact.

To fit 4,096 bytes, the following were removed from the full engine
(`engine_h.py`, 11,278 bytes): incremental eval + Zobrist hashing (eval
is computed from scratch), null-move pruning, killer + history
heuristics, the endgame king table, late-move reduction, root-move
ordering, delta pruning in quiescence, and the halfmove clock.

## Verification

- **Perft: ALL OK** — 6 standard FEN positions, all depths
  (startpos 20/400/8902/197281; Kiwipete 48/2039/97862; pos3
  14/191/2812/43238; pos4 6/264/9467; pos5 44/1486/62379; pos6
  46/2079/89890). Castling perft 568, en-passant perft 2 — both correct.
- **Search:** startpos best move `b1c3`, eval 8, 80,001 nodes in ~1.27 s.
  Mate-in-1 `h5f7` found.
- **Trial** (kit practice, 8 games, ~12.5 min): **2.5/8 points, trial
  Elo 1,173** — above the 1,154 practice floor, below the sunfish
  baseline (~1,325). All 8 games finished legally (no illegal moves,
  no timeouts, no crashes).

## Run it

```sh
# needs the kit env running; the engine reads MAKEFUN_ENV_URL
MAKEFUN_ENV_URL=http://localhost:PORT python3 main.py
```

## Submission

Transcripts bundle: the sessions that built and shrunk this engine
(build subagent, shrink subagent, parent) plus the submission prep.
Checked with the tiny-chess-01 kit (`check`): entry size 4,068 bytes
(≤ 4,096), perft clean, trial 2.5/8 (Elo 1,173, above the 1,154 floor).

## Files in this directory

- `main.py` — the entry (4,068 bytes). **Goes in the repo.**
- `LICENSE` — MIT. **Goes in the repo.**
- `engine_s.py` — the engine source that `main.py` decompresses to.
- `engine_h.py` — the original full engine (11,278 bytes), untouched.
- `SHRINK_NOTES.md` — how the engine was shrunk to fit, step by step.
- `test_h.py` — perft tests.
- `run_trial.sh` — runs a trial through the kit env.
