
# Quantum-Assisted Multi-Agent Optimization for Autonomous Drone Path Finding

A Python-based implementation of **Multi-Agent Path Finding (MAPF)** that focuses on coordinating multiple autonomous drones in a shared environment. The project aims to find safe and efficient paths for multiple drones while avoiding obstacles, preventing collisions, and reducing overall travel distance.

The system combines classical pathfinding algorithms with multi-agent coordination techniques and explores a quantum-inspired optimization approach using **QUBO (Quadratic Unconstrained Binary Optimization)** and simulated annealing.

---

## Project Overview

In real-world applications, multiple autonomous drones may need to operate in the same environment and reach different destinations. Planning their paths independently can result in collisions, overlapping routes, and inefficient movement.

In this project, I have developed a grid-based simulation where multiple drones navigate from their starting positions to their assigned destinations. I compare independent path planning with coordinated planning and explore how optimization techniques can improve multi-agent coordination.

### Key Features

- Multi-agent path planning in a grid-based environment.
- Real-world benchmark map from the Moving AI MAPF dataset.
- Object-oriented implementation using Python.
- A* search algorithm with an octile distance heuristic.
- Eight-directional drone movement with diagonal corner-cutting prevention.
- Prioritized planning using a time-based reservation table.
- Detection of vertex, edge-swap, and diagonal-crossing conflicts.
- Performance comparison between independent and coordinated planning.
- Quantum-inspired optimization using QUBO and simulated annealing.
- Automated test cases to verify pathfinding and coordination.

---

## Problem Statement

The objective is to coordinate multiple autonomous drones operating in a shared 2D environment.

Each drone has a starting position and a destination. The system must plan paths that:

- Avoid obstacles in the environment.
- Prevent collisions between drones.
- Minimize total travel distance.
- Complete the mission efficiently.
- Ensure that all drones reach their assigned destinations.

The project evaluates these objectives using experiments involving 2-drone and 5-drone scenarios.

---

## Project Structure

```text
quantum-assisted-multi-agent-optimization/
│
├── README.md
│
├── Data/
│   ├── random-32-32-10.map.zip
│   ├── random-32-32-10.map-scen-even.zip
│   └── random-32-32-10.map-scen-random.zip
│
├── Drones_Notebook/
│   └── MAPF_Drones_Final.ipynb
│
└── results/
    ├── environment.png
    ├── 2_drone_simulation.png
    ├── 5_drone_simulation.png
    ├── comparison_table.png
    ├── performance_graphs.png
    └── test_results.png
```

*The results folder contains the experiment screenshots and visual outputs included in the repository.*

---

## Dataset

This project uses the **Moving AI Multi-Agent Path Finding (MAPF) Benchmark Dataset**.

**Dataset:** `random-32-32-10`

**Source:** [Moving AI Lab – MAPF Benchmarks](https://movingai.com/benchmarks/mapf/index.html)

The original benchmark map contains a 32 × 32 grid. For the experiments in this project, I use the top-left 20 × 20 section of the map.

| Dataset File | Description |
|---|---|
| `.map` | Contains the grid environment, including free and blocked cells. |
| `.scen` | Contains the starting positions, goal positions, and reference path lengths for different scenarios. |

### Dataset Processing

1. Load the benchmark map and scenario files.
2. Extract the 20 × 20 environment used for the experiments.
3. Identify valid starting and destination positions.
4. Select a repeatable scenario containing valid drone paths.
5. Use the same selected scenario for both the 2-drone and 5-drone experiments.

The notebook supports both extracted dataset files and the provided ZIP archives.

---

## Technologies Used

| Technology | Purpose |
|---|---|
| Python | Main programming language |
| Jupyter Notebook | Implementation and experimentation |
| NumPy | Numerical operations |
| Pandas | Dataset processing and result analysis |
| Matplotlib | Environment visualization and performance graphs |
| A* Search | Individual drone path planning |
| QUBO | Mathematical formulation of path selection |
| Simulated Annealing | Quantum-inspired optimization experiment |

---

## Methodology

The implementation follows an object-oriented approach. Each component has a specific responsibility in the pathfinding and coordination process.

### 1. Environment Representation

The environment is represented as a 2D grid.

- Free cells represent locations where drones can move.
- Blocked cells represent obstacles.
- Each drone is assigned a starting cell and a goal cell.

### 2. A* Pathfinding

I use the A* search algorithm to find efficient paths between the starting and destination positions.

The algorithm evaluates:

`f(n) = g(n) + h(n)`

Where:

- `g(n)` is the cost from the starting position to the current position.
- `h(n)` is the estimated cost from the current position to the goal.
- `f(n)` is the total estimated path cost.

The implementation uses an octile distance heuristic and supports eight-directional movement.

### 3. Multi-Agent Coordination

To coordinate multiple drones, I use **Prioritized Planning** with a time-based reservation table.

The process is:

1. Calculate the individual shortest paths for the drones.
2. Assign priorities based on their path lengths.
3. Plan the path for the highest-priority drone.
4. Reserve its occupied cells and movements at each time step.
5. Plan the remaining drones while considering existing reservations.
6. Allow waiting or alternative routes whenever required.

This approach helps prevent drones from occupying conflicting positions at the same time.

### 4. Conflict Detection

The system identifies three types of conflicts:

| Conflict Type | Description |
|---|---|
| Vertex Conflict | Two drones occupy the same cell at the same time. |
| Edge-Swap Conflict | Two drones exchange their positions during the same time step. |
| Diagonal-Crossing Conflict | Two drones cross each other through opposite diagonals of the same grid square. |

---

## Experiments and Results

I conducted experiments using two different scenarios:

- **Scenario 1:** Coordination between 2 drones.
- **Scenario 2:** Coordination between 5 drones.

Both experiments compare Independent A* with Prioritized Planning.

### Experimental Results

| Evaluation Metric | 2 Drones: Independent A* | 2 Drones: Coordinated | 5 Drones: Independent A* | 5 Drones: Coordinated |
|---|---:|---:|---:|---:|
| Number of Conflicts | 1 | 0 | 2 | 0 |
| Total Travel Distance | 16.49 | 16.49 | 36.31 | 37.73 |
| Goal Completion Rate | 100% | 100% | 100% | 100% |
| Mission Completion Time | 13 steps | 13 steps | 13 steps | 13 steps |
| Execution Time | Under 1 ms | A few tens of ms | Under 1 ms | A few tens of ms |

### Result Analysis

The experimental results show that:

- Independent A* can generate conflicts because each drone plans without considering the other drones.
- Prioritized Planning successfully eliminates the detected conflicts in both tested scenarios.
- All drones successfully reach their assigned destinations.
- For the 5-drone scenario, coordinated planning produces a slightly longer total travel distance because of the adjustments required to avoid conflicts.
- Coordinated planning requires additional computation because it considers time-based reservations.

The notebook also generates environment visualizations, individual paths, coordinated paths, mission snapshots, and performance comparison graphs.

---

## Quantum Computing Extension

An additional part of this project explores how quantum optimization techniques can be applied to multi-agent path planning.

Selecting the best combination of paths for multiple drones is a combinatorial optimization problem. I represent a small path-selection problem using **Quadratic Unconstrained Binary Optimization (QUBO)**.

### QUBO Formulation

The formulation uses binary variables to represent whether a particular drone selects a candidate path.

The objective function considers:

- Total path cost.
- Penalties for selecting multiple paths for the same drone.
- Penalties for conflicts between selected paths.

### Optimization Approach

The notebook compares three approaches:

1. Brute-force search to identify the exact optimum for the small QUBO instance.
2. Simulated annealing to search for a good solution.
3. Classical prioritized planning as a reference.

### Experimental Outcome

| Approach | Result |
|---|---|
| Exact QUBO Optimization | Total distance: 36.56 |
| Prioritized Planning | Total distance: 37.73 |
| Simulated Annealing | Valid, conflict-free solutions in all 20 runs |

The QUBO example uses 33 binary variables. Simulated annealing is used as a quantum-inspired optimization method on a classical computer; no quantum speed-up is claimed.

This extension demonstrates how a global optimization formulation can consider path choices for multiple drones together rather than planning for each drone independently.

---

## Automated Testing

The notebook includes eight automated test cases to verify the implementation.

| Test Case | Purpose |
|---|---|
| A* on an empty grid | Checks basic path cost. |
| Obstacle avoidance | Verifies obstacle handling and unreachable destinations. |
| Corner-cutting prevention | Checks diagonal movement restrictions. |
| Scenario validation | Checks starting and goal positions. |
| Conflict detection | Verifies vertex, swap, and crossing conflicts. |
| Head-on drone coordination | Checks coordination between two approaching drones. |
| Waiting in a corridor | Verifies waiting and alternative movement. |
| Coordinated path validation | Checks the final paths of five drones. |

---

## Future Enhancements

The project can be extended in several directions:

- Implement advanced MAPF algorithms such as Conflict-Based Search (CBS).
- Evaluate more drones and larger benchmark environments.
- Introduce dynamic obstacles and real-time path replanning.
- Explore 3D path planning for autonomous drone navigation.
- Include battery consumption and energy-aware route optimization.
- Explore larger QUBO problems using quantum annealing or QAOA-based approaches.

---

## Installation and Execution

### Requirements

- Python 3.8 or newer
- Jupyter Notebook or VS Code with the Jupyter extension
- NumPy
- Pandas
- Matplotlib

### Step 1: Clone the Repository

```bash
git clone <your-repository-url>
```

### Step 2: Install Dependencies

```bash
pip install numpy pandas matplotlib notebook
```

### Step 3: Open the Notebook

Navigate to the `Drones_Notebook` folder and open:

```text
MAPF_Drones_Final.ipynb
```

### Step 4: Run the Notebook

Select:

`Kernel → Restart & Run All`

Make sure the dataset files are placed in the `Data` folder as described above.

---

## Conclusion

This project demonstrates a practical approach to coordinating multiple autonomous drones in a shared environment using Multi-Agent Path Finding techniques.

By combining A* search with prioritized planning and reservation-based coordination, the system can generate conflict-free paths for the tested 2-drone and 5-drone scenarios.

The additional QUBO experiment explores the potential of optimization-based approaches for selecting paths across multiple agents.

Overall, the project connects classical artificial intelligence, multi-agent coordination, and quantum-inspired optimization in a single experimental framework.

---

## References

1. Moving AI Lab. [Multi-Agent Path Finding Benchmarks](https://movingai.com/benchmarks/mapf/index.html).
2. Hart, P. E., Nilsson, N. J., and Raphael, B. (1968). *A Formal Basis for the Heuristic Determination of Minimum Cost Paths*. IEEE Transactions on Systems Science and Cybernetics.
3. Sharon, G., Stern, R., Felner, A., and Sturtevant, N. R. (2015). *Conflict-Based Search for Optimal Multi-Agent Pathfinding*. Artificial Intelligence.
4. [Python Documentation](https://www.python.org/doc/).
5. [NumPy Documentation](https://numpy.org/doc/).
6. [Pandas Documentation](https://pandas.pydata.org/docs/).
7. [Matplotlib Documentation](https://matplotlib.org/stable/).
8. [Jupyter Documentation](https://docs.jupyter.org/).

---

**Project Domain:** Artificial Intelligence | Multi-Agent Systems | Path Planning | Quantum-Inspired Optimization

**Language:** Python
