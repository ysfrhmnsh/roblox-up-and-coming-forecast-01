# Roblox Up-and-Coming Forecast — BUILD NOTES

Bounty: make.fun `roblox-up-and-coming-forecast-01` ($500 USDC; entries close
2026-11-13 06:00 UTC). Entry is `main.py` in this directory. Build + test only —
no repo created, nothing submitted, no accounts/keypairs.

## Result (practice window, scored locally with the exact RULES scorer)

| Forecast | Log score (10 reads) | Top-10 hits/read | Agreement/read |
|---|---|---|---|
| House baseline (persistence) | **−43.512039** | 6.4 | 53.5 |
| `main.py` (final) | **−33.871227** | 6.4 | 54.7 |

**Beats the baseline by +9.64 points.** The score was reproduced exactly by my
local scorer (baseline −43.512039 matches the RULES sheet to 6 decimals, so the
scorer is validated).

Second validation split (train on archive reads before 2026-11-04, score the
next 72h, 11 reads): `main.py` config −38.31 vs baseline −48.63 (**+10.3**).
The config choice was the best on *both* splits, not just the practice window.

Caveat: the house AI-model forecast is the other bar and its score is unknown.
This entry clears the persistence bar by a wide margin; whether it clears the
house model is unknowable locally.

## Approach — Monte Carlo rank simulation (`main.py`, stdlib only)

Each game has a latent rank-value that each hour mean-reverts toward a target
with band-dependent Gaussian noise:

- **Target** = last rank + shrunk recent rank velocity × 36h (velocity shrunk
  ×0.2 — the measured regression slope of velocity on its next value is 0.196).
  Games climbing fast (velocity < −0.5/h) get riser dynamics: target =
  rank + velocity × 24h, faster reversion.
- **Noise σ/hour by rank band** (measured on the 51-read public archive):
  top 10: 0.82, 11–30: 2.10, 31–60: 3.32, 61+: 3.85.
- **New games** spawn at the bottom (Poisson 0.8/hour, entry rank ~60–97);
  45% are "risers" climbing exponentially toward a top-10 target
  (25% of risers target places 1–2, rest 3–10; reversion 0.09/h — this matches
  the observed riser path: enter ~rank 55, 28 at +12h, 13 at +24h, 9 at +48h).
- **Aging-out**: games leave 28 days after publishing (`details.created` when
  present, else first-seen minus 3 days).
- 2,000 simulations × 72 hours; forecast = time-averaged marginal
  P(game at place k). Residual mass is left **unnamed** (0.11 at place 1 →
  0.26 at place 10) so brand-new games share it — the log score punishes
  confident wrong calls far more than it rewards confident right ones.
- `top10` = greedy max-probability assignment (each game/place used once).

## What the data showed (and what did NOT work)

- Rank is only weakly driven by raw player counts (rank-vs-playing concordance
  0.65; growth metrics ~0.50 = random). Playing-momentum correlation with next
  rank change is −0.06 — **no signal**; rank history is the signal.
- Top-10 membership decays: a close top-10 game is still top 10 in 80% of
  reads at +12h, 67% at +24h, 48% at +48h, 35% at +72h. Same game at same
  place: 28% at +12h, 3% at +72h — persistence is badly overconfident, which
  is why the baseline scores −43.5.
- Brand-new games take ~8% of top-10 slots per 72h window (pooled over 50
  windows), mostly entering at places 7–10 but occasionally shooting to #1
  ("Do Not Stop." was unseen in history, hit #1 within ~2 days). The model
  deliberately hedges above the empirical rate (log-score asymmetry: a missed
  newcomer at #1 costs far more than slightly diluted named probabilities).
- Pure random-walk ranks scored −38.55; mean reversion −36.80; riser dynamics
  + tuning −33.87. Each step was checked on both validation windows.

## Verification of `main.py`

- Two runs on the kit practice input: **byte-identical output** (deterministic).
- Runtime ~55–60s per run (limit 300s), 1 CPU, stdlib only.
- Output 17,979 bytes (limit 1,048,576), exactly two keys (`probabilities`,
  `top10`), 315 rows, per-place sums 0.74–0.90 (≤ 1), all p in (0, 1).
- Reads window/slots/output name from `task.json` (not hardcoded); handles
  both archive snapshots (no `details`) and house-recorder snapshots (hourly,
  with `details.created`).

## What to try next (if time before the deadline)

1. **Re-tune on the final history** (it will include hourly house reads with
   `details.created` — velocity estimates get much finer; the same code runs).
2. Try a per-place unnamed-share calibration: the pooled empirical newcomer
   rate is U-shaped (places 7–10 ~10–14%, places 1–3 ~2–7%); the sim's
   unnamed share already rises toward place 10, but an explicit cap/floor
   per place could be tested on both splits.
3. Try modelling the chart size (92–100) explicitly instead of the fixed
   median; marginal.
4. Do NOT chase the house-model bar blindly — no local signal for it exists;
   the entry is already well past persistence.

## Files

- `main.py` — the entry (Python 3.13, stdlib only).
- `roblox-up-and-coming-forecast-01-kit/` — downloaded kit (RULES.txt, kit.json,
  starter, harness source, input/).
- `work/` — local scorer (`scorer.py`, reproduces baseline exactly),
  simulators (`sim.py`, `sim2.py`, `sim3.py`), split test, grids, analysis
  scripts. Not part of the submission.
