# Quantum-Assisted Multi-Agent Optimization: Drone Path Finding

Coordinating several autonomous drones in a shared 2D grid so that every drone reaches its target without hitting obstacles or other drones, using a short distance and a short mission time. The project uses a real benchmark map, compares an uncoordinated baseline with a coordinated planner, and adds a small quantum-inspired (QUBO + simulated annealing) experiment.

**Author:** [Your name]  |  **Course / Assignment:** [Course name]  |  **Language:** Python 3 (Jupyter Notebook)

---

## Table of contents

1. [At a glance](#at-a-glance)
2. [Problem statement](#problem-statement)
3. [Project structure](#project-structure)
4. [Dataset](#dataset)
5. [Quick start](#quick-start)
6. [How the solution works](#how-the-solution-works)
7. [Experiments and results](#experiments-and-results)
8. [Quantum computing extension](#quantum-computing-extension)
9. [Test cases](#test-cases)
10. [Limitations and future work](#limitations-and-future-work)
11. [Troubleshooting](#troubleshooting)
12. [Requirements](#requirements)

---

## At a glance

| Item | Summary |
|---|---|
| Problem | Multi-Agent Path Finding (MAPF) for drones on a grid |
| Environment | 20 x 20 window of the Moving AI map `random-32-32-10` (about 12 % obstacles) |
| Agents | 2 drones (Scenario 1) and 5 drones (Scenario 2) |
| Baseline | Independent A\* (each drone ignores the others) |
| Coordination | Prioritized planning with a time-based reservation table |
| Key result | Conflicts drop to 0 in both scenarios, all drones reach their goals (100 %), mission time unchanged |
| Quantum part | Path selection written as a QUBO (33 binary variables) and solved with simulated annealing |
| Verification | 8 automatic test cases, all expected to print PASS |

---

## Problem statement

A group of drones operates in a shared environment. Each drone has a start cell and a target cell. The goal is to move all drones to their targets while:

- avoiding obstacles,
- avoiding collisions between drones,
- minimizing the total travel distance,
- minimizing the mission completion time.

Main challenges: collisions, congestion in narrow gaps, conflicting paths, and scalability (the joint search space grows exponentially with the number of drones).

---

## Project structure

```
quantum-assisted-multi-agent-optimization/
|-- README.md
|-- Data/
|   |-- random-32-32-10_map
|   |-- random-32-32-10_map-scen-even
|   `-- random-32-32-10_map-scen-random
`-- Drones_Notebook/
    `-- MAPF_Drones_Final.ipynb           
```

---

## Dataset

**Source:** Moving AI Lab, Multi-Agent Pathfinding benchmarks. Map: `random-32-32-10` (32 x 32 grid, 10 % random obstacles).

| File | Content |
|---|---|
| `random-32-32-10.map` | the grid; `.` is a free cell and `@` is an obstacle |
| `random-32-32-10-even-1` to `-25.scen` | start cell, goal cell and optimal length of many drone trips; "even" means trips of all lengths are represented |

**How the notebook uses the data**

1. Reads the full 32 x 32 map and cuts the **top-left 20 x 20 window** as the environment.
2. Reads the 25 `even` scenario files and keeps only entries whose start and goal are both inside the window, on free cells.
3. Picks the scenario file with a fixed, repeatable rule: the first `even` file (in numeric order) where the independent A\* paths of the first 2 drones collide. With the provided data this is `random-32-32-10-even-2.scen`.
4. Uses the first 2 valid drones of that file for Scenario 1 and the first 5 for Scenario 2.

The `random` scenario files and the benchmark's optimal-length column are not used. The notebook accepts either the extracted files or the ZIP files inside `Data/` (it extracts ZIPs by itself when needed).

---

## Quick start

1. Keep the folder structure shown above.
2. Install the requirements:
   ```
   pip install numpy pandas matplotlib notebook
   ```
3. Open `Drones_Notebook/MAPF_Drones_Final.ipynb` in Jupyter or VS Code.
4. Select **Kernel -> Restart & Run All**.

**What a correct run looks like**

- The data cell prints the 32 x 32 map size, the 20 x 20 window and 25 scenario files.
- The scenario-selection cell prints `Scenario files with at least 5 valid drones: 25 of 25` and `Selected scenario file: random-32-32-10-even-2.scen`.
- The test cell prints 8 lines that start with `PASS`.

---

## How the solution works

### Classes (object-oriented design)

| Class | Responsibility |
|---|---|
| `Grid` | the environment: free cells, 8-direction moves, move costs, octile heuristic |
| `Drone` | one agent: id, start, goal, planned path, distance, arrival time |
| `Planner` | A\* search; with reservations it searches in (cell, time) and may wait |
| `Reservations` | shared table of cells and moves already taken at each time step |
| `Coordinator` | independent planning (baseline) or prioritized planning (coordination) |

### Movement rules

- Time is discrete and all drones move at the same moment.
- 8 directions: straight step costs 1, diagonal step costs sqrt(2), waiting costs 0.
- A diagonal step cannot cut a corner between two obstacles.
- After arriving, a drone parks at its goal and keeps that cell reserved.

### Algorithms

- **A\*:** expands the state with the smallest `f = g + h`, where `g` is the cost so far and `h` is the octile distance `max(dx, dy) + (sqrt(2) - 1) * min(dx, dy)`. The heuristic never overestimates, so the path is optimal for a single drone.
- **Independent A\*:** every drone runs A\* alone. Fast, but conflicts are possible.
- **Prioritized planning:** drones are sorted by the length of their solo shortest path (longest first). Each drone plans around the reservations of the drones before it, using detours or waiting, and then adds its own path to the reservation table.

### Conflict types counted

| Type | Meaning |
|---|---|
| Vertex | two drones in the same cell at the same time step |
| Swap | two drones exchange cells during the same step |
| Crossing | two drones cross on the two diagonals of the same 2 x 2 square |

### Performance metrics

Total travel distance, number of conflicts (split by type), percentage of drones reaching their goal, mission completion time (time steps), and algorithm execution time (ms).

---

## Experiments and results

Both methods are run on Scenario 1 (2 drones) and Scenario 2 (5 drones), taken from the same scenario file.

| Metric | Scenario 1: Independent A\* | Scenario 1: Prioritized | Scenario 2: Independent A\* | Scenario 2: Prioritized |
|---|---|---|---|---|
| Conflicts | 1 | **0** | 2 | **0** |
| Total travel distance | 16.49 | 16.49 | 36.31 | 37.73 |
| Agents reaching goal | 100 % | 100 % | 100 % | 100 % |
| Completion time (steps) | 13 | 13 | 13 | 13 |
| Execution time | under 1 ms | a few tens of ms | under 1 ms | a few tens of ms |

**Interpretation**

- Prioritized planning removes every conflict, and all drones still reach their goals.
- Waiting costs no distance, so total distance stays the same or grows only slightly when a detour is chosen.
- The price of safety is some waiting time and a longer planning time, because the search also has a time dimension.
- Execution times vary slightly from run to run.

**Visual outputs in the notebook:** environment map, start-cell heat map, start and goal markers, independent paths, coordinated paths with conflict locations marked, time snapshots of the 5-drone mission, and bar charts of all metrics.

---

## Quantum computing extension

**Which part could benefit?** Choosing the best combination of paths for all drones together is a combinatorial optimization problem. Problems of this form can be given to quantum annealers or to QAOA on gate-based quantum computers.

**QUBO formulation.** One binary variable `x(i,p)` means "drone i uses candidate path p". The energy to minimize is:

```
E(x) =  sum over (i,p) of  cost(i,p) * x(i,p)                    path cost (distance + small time penalty)
      + P * sum over i of ( sum over p of x(i,p) - 1 )^2         each drone picks exactly one path
      + W * sum of  c(i,p,j,q) * x(i,p) * x(j,q)                 penalty for conflicts between chosen paths
```

`c(i,p,j,q)` is the number of conflicts between candidate path p of drone i and candidate path q of drone j. `P` and `W` are penalty weights that must be larger than the path costs.

**Example in the notebook.** 5 drones, up to 3 routes and 3 waiting options per drone, giving 33 binary variables and 8 748 possible combinations. It is solved with:

- brute force (exact optimum),
- simulated annealing (quantum-inspired, runs on a normal computer, 20 runs),
- and compared with prioritized planning.

**Outcome.** The exact QUBO optimum has a slightly shorter total distance (36.56) than prioritized planning (37.73), because it chooses all paths together. Simulated annealing gave valid, conflict-free plans in all 20 runs, close to the exact optimum but not always equal to it.

**Advantages.** Global view of all drones, new rules are added as extra penalty terms, and the same QUBO can be sent to different solvers.

**Limitations.** The number of variables grows quickly with drones and candidate paths; penalty weights need tuning; candidate paths must first be generated by a classical algorithm; annealing gives no guarantee. **No quantum speed-up is claimed.** At this size, A\* with prioritized planning is faster and simpler.

---

## Test cases

| Test | What it checks |
|---|---|
| `test_astar_empty_grid` | cost of a corner-to-corner path equals 4 diagonal steps |
| `test_astar_avoids_obstacles_and_unreachable` | path goes around a wall; unreachable goal returns no path |
| `test_no_corner_cutting` | diagonal squeeze between two obstacles is forbidden |
| `test_scenario_entries_are_valid` | experiment drones start and end on free cells inside the window |
| `test_conflict_detection` | vertex, swap and crossing conflicts are counted correctly |
| `test_head_on_drones_are_coordinated` | two drones flying head-on collide when independent and do not when coordinated |
| `test_waiting_in_a_corridor` | a drone steps into a side pocket so another can pass |
| `test_coordinated_paths_are_valid` | final 5-drone paths have legal moves, correct start and goal, and 0 conflicts |

---

## Limitations and future work

**Limitations**

- Prioritized planning is a heuristic: it is neither optimal nor complete, and a bad priority order can make a drone fail.
- The world is perfect and discrete: no wind, delays, drone size or communication errors.
- Only 2 and 5 drones were tested on one 20 x 20 window and one selected scenario file.
- The QUBO example is small and was not run on real quantum hardware.

**Future work**

- Conflict-Based Search (CBS) and other optimal MAPF solvers, several priority orders, re-planning during flight.
- Larger maps, more drones, 3D movement, battery limits.
- Larger QUBOs with tuned penalty weights, tested on a quantum annealer or a QAOA simulator.

---

## Troubleshooting

| Problem | Cause and fix |
|---|---|
| Message `Put the 3 ZIP files ... into a folder called 'Data'` | The dataset was not found. Place the ZIP files or the extracted `.map` and `.scen` files in `Data/`, next to `Drones_Notebook/`. |
| `NameError` such as `env`, `Planner` or `results` is not defined | Cells were run out of order. Use Kernel -> Restart & Run All. |
| First line of the selection cell shows a number lower than 25 | Some scenario files have too few valid drones inside the window. The notebook skips them automatically and still runs. |
| `KeyError: 'Index'` in the scenario-selection cell | An old copy of the cell is still in the notebook. Use the current version of `pick_agents` (it selects rows with `df.iloc`). |
| Different timing numbers | Execution time depends on the computer and changes slightly each run. All other numbers are repeatable. |

---

## Requirements

- Python 3.8 or newer
- numpy, pandas, matplotlib
- Jupyter Notebook (or VS Code with the Jupyter extension)

**Data source:** Moving AI Lab MAPF benchmarks (https://movingai.com/benchmarks/mapf/index.html).
