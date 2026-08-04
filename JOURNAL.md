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

---

<!--
Template for a new entry — copy this block, fill it in, keep newest at the bottom:

## Day N — <short title> (Phase X)

<5 or so sentences on what I did, why, and any decision or gotcha worth
remembering.>
-->
