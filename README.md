# NBA Individual Performance from 2010-2025

Goal: Rate the 10 best, 10 most average, and 10 worst individual NBA player seasons (regular season only, ≥500 minutes, seasons 2010–11 onward) using only the provided dataset.

---

## Project Overview

- Cleans game logs, builds player–season aggregates, computes efficiency and per-minute rates, and produces three lists required by the prompt.

---

## Dataset & Inputs

- link: https://www.kaggle.com/datasets/eoinamoore/historical-nba-data-and-player-box-scores/data
- This notebook expects game-level player box scores (e.g., PlayerStatistics.csv) with columns like:

- Features: personId, firstName, lastName, gameId, gameDate, gameType, gameLabel, gameSubLabel, playerteamName, opponentteamName
- Minutes: numMinutes (string-like MM.SS or MM:SS)
- Box score: points, assists, reboundsTotal, steals, blocks, turnovers, fieldGoalsAttempted, fieldGoalsMade, freeThrowsAttempted, freeThrowsMade, threePointersAttempted, threePointersMade.

---

## Scope 

- Regular season only (keeps In-Season Tournament games; excludes All-Star events).

- Seasons ≥ 2010–11.

- Player-season minutes ≥ 500 (after cleaning and conversion).

---
## Preprocessing

- Filter games

Keep gameType = regular season.

Remove All-Star Weekend rows by gameLabel/gameSubLabel patterns (All-Star, Rising Stars, Skills, 3-Point, Dunk).

Keep In-Season Tournament (NBA Cup) games — they count toward the regular season.

- Minutes cleanup

Convert numMinutes from MM.SS or MM:SS → decimal minutes min_dec.
Example: 30.46 → 30 + 46/60 = 30.7667.

Drop data errors: rows with stats > 0 but minutes = 0/blank.

DNPs (0 minutes, 0 stats) are not used for analysis.

- Season labeling

Create season string YYYY-YY (e.g., 2018-19) based on game date (season starts in October).

- Minutes gate

Aggregate minutes per player-season and keep ≥ 500 total minutes (multi-team seasons aggregated).

---

## Features & Metrics

- For each qualified player-season:

Shooting efficiency:

True Shooting % (TS%) = PTS / [2 × (FGA + 0.44 × FTA)]

WS_Proxy :

Offensive_Contribution = PTS+0.7⋅AST+0.7⋅REB−0.4⋅TOV−0.4⋅PF
Defensive_Contribution = STL+BLK+0.3⋅DRB

WS_proxy = (Offensive_Contribution + Defensive_Contribution) * 48 / min

PER_Proxy = ​(PTS+REB+AST+2⋅STL+2⋅BLK−(FGA−FGM)−0.7⋅(FTA−FTM)−TOV) * 48 / min

---

## Summary measures

PER-like box composite: Interpretable approximation from positive box stats (PTS, REB, AST, STL, BLK) minus missed shots & turnovers, scaled per minute (not the official Hollinger PER).

WS/48 (Win Shares per 48): impact rate derived from win contributions (if Win Shares are present; otherwise omitted).

The notebook standardizes features within each season using z-scores (optionally winsorized at ±3σ) to keep comparisons fair across years and robust to outliers.

---

## Composite Scoring System

To produce a single interpretable ranking for player seasons, I combined multiple metrics into a composite score. Each metric was first normalized (using Min-Max scaling) so they were on the same scale, and then weighted based on its importance to evaluating overall player impact.

The weights were chosen to balance scoring, efficiency, playmaking, and defense fairly across different roles:

Metric	Weight
Points per Game (PPG)	0.15
Rebounds per Game (RPG)	0.12
Assists per Game (APG)	0.12
Steals per Game (SPG)	0.10
Blocks per Game (BPG)	0.10
True Shooting % (TS%)	0.15
Win Shares Proxy (WS/48)	0.13
PER Proxy (per-48)	0.13

The final composite score is a weighted sum of these normalized values:

Composite Score = Sum ( Weights * Metric)

This ensures that no single metric dominates the ranking and that the evaluation reflects a balance of efficiency, volume, and defensive impact.

---
