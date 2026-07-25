# Soccer Analytics & Scouting Platform

A portfolio project built around a **from-scratch expected goals (xG) model**
and a **match/player performance predictor**, trained on StatsBomb open event data.

## What I'm building
- **ML**: my own xG model (logistic regression → gradient boosting) plus a match
  outcome predictor, with full feature engineering and model comparison.
- **Backend**: a FastAPI service that serves predictions and player/match stats.
- **Frontend**: a React + TypeScript dashboard with shot maps, xG timelines, and a
  "predict this matchup" tool.
- **Database**: PostgreSQL storing matches, players, teams, and shot events —
  including both StatsBomb's xG and my model's xG for benchmarking.

## Status
🚧 Phase 0 — project setup. Building in phases; see commit history.

## Stack
Python · pandas · scikit-learn · XGBoost · FastAPI · SQLAlchemy · Alembic ·
PostgreSQL · React · TypeScript · Vite
