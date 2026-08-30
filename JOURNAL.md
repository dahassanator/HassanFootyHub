# Development Journal

A running log of what I worked on each session building the Soccer Analytics &
Scouting Platform. Newest entries at the bottom. Pairs with the commit history —
the commits show *what changed*, this shows *what I was thinking*.

---

## Day 1 — Project scaffolding (Phase 0)

I set up the foundation for the soccer analytics platform by creating the GitHub
repository and cloning it locally. I built out the full monorepo file skeleton,
establishing separate top-level directories for the ML experimentation (`ml/`),
the data pipeline, the FastAPI backend, and the React frontend, so each subsystem
has a clear home from the start. I wrote the README to describe the project's goal
— a from-scratch xG model and match predictor built on StatsBomb open data — along
with the phased build plan and the full tech stack. I added a `.gitignore`
configured to keep pulled data, model artifacts, virtual environments, and
`node_modules` out of version control. Finally, I made the first commit and pushed
it, completing Phase 0 with the repo structure in place before writing any real
code.

## Day 2 — Data exploration notebook (Phase 1)

I built the `01_data_exploration` notebook to get familiar with the StatsBomb data
and confirm I understood its structure before modeling. I pulled the data
hierarchy with `statsbombpy` — competitions, then matches, then events — and
learned that everything is keyed on `(competition_id, season_id)` and then
`match_id`. I inspected the wide event schema, isolated the Shot events, and
identified the columns that matter for xG: the outcome (my future target), body
part and shot type (features), and StatsBomb's own xG value (my benchmark). I
unpacked the `location` field into separate `x`/`y` columns and internalized the
coordinate system (120×80 pitch, goal at x=120, y=40), then plotted shot maps with
`mplsoccer` to *see* how chance quality clusters near goal. I finished by looping
over every match to assemble a tournament-wide shot dataset and running a
calibration sanity check, confirming that StatsBomb's mean xG lands close to the
actual goal rate.

## Day 3 — Feature engineering notebook (Phase 1)

I built the `02_xg_features` notebook, which turns raw shot rows into a clean
numeric feature matrix ready for modeling. The intellectual core was the geometry:
I computed each shot's distance to goal by Pythagoras and its shot angle — the
angular width of the goal mouth from the shooter's position — using `arctan2` for
numerical robustness. I validated both features by bucketing shots and confirming
goal rate falls monotonically with distance and rises with angle, treating the
minor inversions in low-count buckets as expected sampling noise rather than bugs.
I one-hot encoded the categorical features (body part and shot type), reasoned
through why penalties are a distinct process best handled by their own flag, and
assembled the final `X` and `y`. I also explored deriving a strong-foot/weak-foot
feature, learning that footedness isn't in the open data and must be inferred from
a player's body-part usage — and that because it depends on global player state, it
would eventually require a precomputed lookup rather than living in a stateless
feature function.

## Day 4 — xG model training (Phase 2)

I built the `03_xg_model` notebook, where I finally trained and compared real xG
models on the feature matrix from 02. I used a three-way stratified split — train
to fit, a small validation set purely to time XGBoost's early stopping, and a test
set I never touched until the final read — so the headline metrics stay honest and
nothing leaks in from evaluation. I fit a logistic regression baseline inside a
`StandardScaler` pipeline first, then XGBoost with early stopping, and deliberately
skipped any class rebalancing: for xG I care about calibrated probabilities, not
classification, and reweighting the classes would distort exactly the numbers I'm
trying to trust. The result that shouldn't have surprised me but did — logistic
regression narrowly won on every metric (log loss 0.2393 vs 0.2401, ROC-AUC 0.865
vs 0.858), with both models beating the base-rate floor by roughly 30% — and the
lesson is that with only nine features and a smooth, monotonic relationship between
distance/angle and goal probability, there's nothing nonlinear for the trees to
exploit, so the simpler model wins and I can explain precisely why. I checked
calibration against StatsBomb's own xG and got a 0.984 per-match correlation on
total xG, while noting the honest caveat that my summed xG landing near the actual
goal count is mostly a property of intercept-fit logistic regression, not proof I
beat StatsBomb. Finally I saved the winning pipeline with `joblib` — the scaler
travels inside the artifact, so there's no training-serving skew at inference —
and logged the one real gotcha: I had to re-pull the raw shots to recover
`shot_statsbomb_xg` and `match_id`, which notebook 02 never wrote to the CSV.

---

<!--
Template for a new entry — copy this block, fill it in, keep newest at the bottom:

## Day N — <short title> (Phase X)

<5 or so sentences on what I did, why, and any decision or gotcha worth
remembering.>
-->
