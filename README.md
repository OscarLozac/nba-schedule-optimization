# NBA Schedule Optimization


## Overview

The NBA is planning its 2025/2026 season schedule. Starting from a preliminary schedule (`games.csv`), this project uses integer programming to analyze, reproduce, and improve the schedule under a set of structural and operational constraints.

The project is structured in four questions:

| Question | Description | Result |
|---|---|---|
| Q1 | Extract schedule statistics (home/away dates, matchup counts) | Completed |
| Q2 | Integer program whose feasible solutions are all valid schedules | Feasible — 128 games scheduled |
| Q3 | Add time zone travel constraints — feasibility analysis | Infeasible under strict threshold |
| Q4 | Minimize maximum travel distance across all teams | 21.2% improvement over original |

---

## Problem Description

Given a preliminary NBA schedule, a feasible schedule must satisfy, for each team:
- Home games occur exactly on the same dates as in the original schedule
- Away games occur exactly on the same dates as in the original schedule
- The number of home games against each opponent is preserved
- The number of away games against each opponent is preserved

Beyond feasibility, two additional problems are explored: a travel constraint based on time zone differences between consecutive games, and an optimization objective minimizing the maximum distance traveled by any team.

---

## Results

### Question 1 — Schedule Statistics

Extracted for all 16 teams: home dates, away dates, and home/away matchup counts against each opponent. These statistics serve as parameters for the integer programming model in Question 2.

### Question 2 — Feasibility Model

Decision variable: x[i, j, d] = 1 if team i hosts team j on date d, 0 otherwise.

The model enforces home/away date constraints and matchup frequency constraints for every team pair. The CBC solver (via PuLP) finds a feasible solution in under 0.05 seconds.

- Status: Optimal
- Games scheduled: 128
- Schedule saved to `schedule_q2.csv`

### Question 3 — Time Zone Constraints

Each arena is assigned a UTC offset. For any three consecutive games played by a team, the sum of absolute time zone differences between consecutive games must be strictly less than 4.

Result: infeasible under the strict threshold. Three violations were identified in the original schedule:

- Brooklyn Nets: New York (-5) → Los Angeles (-8) → Phoenix (-7), cumulative diff = 4
- Los Angeles Lakers: Los Angeles (-8) → Dallas (-6) → Los Angeles (-8), cumulative diff = 4
- Los Angeles Lakers: Dallas (-6) → Los Angeles (-8) → Chicago (-6), cumulative diff = 4

Sensitivity analysis showed that the constraint becomes feasible when the threshold is relaxed to 6, or when it is applied only to consecutive away games (road trips).

### Question 4 — Minimize Maximum Travel Distance

Using geographic coordinates for all 16 arenas, the model minimizes the maximum total distance traveled by any team across the season (minimax formulation).

- Original schedule: max distance = **37,160 km** (Golden State Warriors)
- Optimized schedule: max distance = **29,292 km**
- Improvement: **21.2%**
- Solve time: 42 seconds
- Optimized schedule saved to `schedule_q4_minmax_dist.csv`

---

## Repository Structure

```
nba-schedule-optimization/
│
├── data/
│   └── games.csv                      — preliminary schedule (16 teams, 128 games)
│
├── NBA_Schedule_Project.ipynb         — full code: Q1 to Q4 with outputs
├── Project_2.pdf                      — written report
├── schedule_q2.csv                    — feasible schedule from Q2
├── schedule_q4_minmax_dist.csv        — optimized schedule from Q4
├── .gitignore
└── README.md
```

---

## Methodology

### Question 2 — Integer Programming Formulation

Sets: T = teams, D = dates.

Decision variable: x[i, j, d] in {0, 1} — team i hosts team j on date d.

Constraints:
- For each team i and home date d: sum over j of x[i,j,d] = 1
- For each team i and non-home date d: sum over j of x[i,j,d] = 0
- For each team i and away date d: sum over j of x[j,i,d] = 1
- For each team pair (i,j): sum over d of x[i,j,d] = h[i,j] (home matchup count)
- For each team pair (i,j): sum over d of x[j,i,d] = a[i,j] (away matchup count)

### Question 3 — Time Zone Linearization

For away games, the time zone of team i on date d is expressed as a linear combination of opponents' time zones weighted by the binary variables. Absolute differences are linearized using auxiliary variables and standard two-inequality encoding. The constraint sum(diff1, diff2) <= 3 is added for every triple of consecutive games.

### Question 4 — Minimax Travel Distance

For each team, travel distance between consecutive games is expressed as a linear function of the binary schedule variables using precomputed arena-to-arena distances (Haversine formula). A global variable M >= max(total distance per team) is minimized.

---

## Data

`games.csv` contains the preliminary 2025/2026 NBA season schedule with fields: Date, Visitor, Home, Arena, attendance, and tip-off time. The dataset covers 16 teams over the period November 2025 to December 2025.


