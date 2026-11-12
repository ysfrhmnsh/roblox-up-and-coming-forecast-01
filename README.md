# Roblox Up-and-Coming top 10 forecast

Entry for the make.fun bounty `roblox-up-and-coming-forecast-01` ($500 USDC).

`main.py` (Python 3.13, standard library only) reads hourly snapshots of the
Roblox Up-and-Coming chart from `/input` and writes `/output/forecast.json`:
for each of the top 10 places, a probability for each game it names being at
that place, plus its guess at the top 10 in order, for every hour of the 72
hours after entries close.

Approach: Monte Carlo rank simulation. Each game has a latent rank-value that
mean-reverts toward a target extrapolated from its recent rank velocity, with
band-dependent hourly noise measured from the chart archive; new games spawn
below the chart (a fraction are "risers" climbing exponentially toward the
top 10); games age out 28 days after publishing. Probabilities are kept
honest and spread — the log score punishes confident wrong calls far more
than it rewards confident right ones.

Practice window (local scorer, validated by reproducing the house baseline
exactly): **−33.87 vs baseline −43.51 (+9.64)**. Independent second split:
−38.31 vs baseline −48.63 (+10.3). Deterministic, ~65 s per run.

See `BUILD_NOTES.md` for the full approach, calibration numbers, and scores.
