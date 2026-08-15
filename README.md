# 🧭 Maze Pathfinding — BFS · DFS · A\* · Dijkstra

![Python](https://img.shields.io/badge/Python-3.8%2B-3776AB?logo=python&logoColor=white)
![Dependencies](https://img.shields.io/badge/dependencies-none%20(stdlib)-brightgreen)
![GUI](https://img.shields.io/badge/GUI-Tkinter-orange)
![Course](https://img.shields.io/badge/HUST-Intro%20to%20AI%202023-blueviolet)

Four classical search algorithms, implemented from scratch against one shared maze abstraction and
animated step by step in Tkinter — so you can **watch** the difference rather than only read the
complexity bounds.

> Term project for **Introduction to Artificial Intelligence**, Hanoi University of Science and
> Technology (HUST), 2023.

| | Algorithm | Behaviour | Shortest path? |
|:-:|---|---|---|
| 🌊 | **BFS** | floods the maze level by level | ✅ always |
| 🕳️ | **DFS** | dives down one corridor, backtracks at dead ends | ❌ 1 / 30 mazes |
| ⭐ | **A\*** | goal-biased BFS, ordered by `f = g + h` | ✅ always — and ~20% less search |
| 💰 | **Dijkstra** | same guarantee, plus per-cell terrain costs | ✅ always |

---

## 🚀 Quick start

```bash
git clone https://github.com/tant2tls/Intro-AI-Find-Path-in-maze-using-Astar-BFS-DFS-DIJISTRA.git
cd Intro-AI-Find-Path-in-maze-using-Astar-BFS-DFS-DIJISTRA

python BFSDemo.py         # 🌊 breadth-first          (10×16)
python DFSDemo.py         # 🕳️ depth-first            (20×16)
python aStar.py           # ⭐ A* + Manhattan h       (12×14)
python dijkstraMaze.py    # 💰 Dijkstra + cost cells  (20×20)

python DFSvsBFS.py        # side-by-side: path length, cells searched, wall time
python compare3.py        # BFS vs DFS vs A* on one 30×30 maze
```

🖥️ **A display is required** — every entry point opens a Tkinter window; there is no headless or CLI
mode. Maze size is edited in each file's `__main__` block, and mazes are random on every launch, so
no two runs are identical. On Debian/Ubuntu, `tkinter` is packaged separately:
`sudo apt install python3-tk`.

📓 **The written report** is the course deliverable, in two versions — run either from the repository
root:

| Notebook | Language |
|---|---|
| `G6_TÌM_ĐƯỜNG_ĐI_TRONG_MÊ_CUNG.ipynb` | Vietnamese (the graded submission) |
| `report.ipynb` | English translation — text only; every code cell is token-for-token identical |

---

## 📂 What's in here

| File | Role |
|---|---|
| 🗺️ **`pyamaze.py`** | **The environment**: maze generation + all rendering. A modified fork of [MAN1986/pyamaze](https://github.com/MAN1986/pyamaze). **Contains no solver.** |
| 🌊 `BFSDemo.py` | Breadth-first search — `BFS(m, start=None)` |
| 🕳️ `DFSDemo.py` | Depth-first search — `DFS(m, start=None)`; also marks branch points |
| ⭐ `aStar.py` | A\* + three heuristics (`h_mahattan`, `h_euclide`, `h_diagonal`) |
| 💰 `dijkstraMaze.py` | Dijkstra with variable-cost "hurdle" cells — `dijkstra(m, *hurdles, start=None)` |
| 🔬 `DFSvsBFS.py`, `compare3.py` | Comparison harnesses (2-way, 3-way) |
| 🔬 `aStarHeuristicComparison.py`, `test4.py` | Older comparison scripts |
| 📓 `G6_TÌM_ĐƯỜNG_ĐI_TRONG_MÊ_CUNG.ipynb`, `report.ipynb` | The report (Vietnamese / English) |

The four algorithm modules are the reusable core; the rest are thin scripts that import them and
animate several agents at once. `aStar.py` is the reference A\* — prefer it for anything
load-bearing. 

---

## 🗺️ How the maze is represented

Two data structures are enough to read every solver here.

1. **Cells are 1-indexed `(row, col)` tuples** — `m.grid` lists them all.
2. **`m.maze_map` is the adjacency map**, recording which walls are open (`1` = passable):

```python
m.maze_map[(3, 5)]        # {'E': 1, 'W': 0, 'N': 1, 'S': 0}
```

`E`/`W` change the **column**, `N`/`S` the **row** — so every solver shares one inner loop:

```python
for d in 'ESNW':
    if m.maze_map[currCell][d] == True:
        if d == 'E': child = (currCell[0], currCell[1] + 1)
        if d == 'W': child = (currCell[0], currCell[1] - 1)
        if d == 'N': child = (currCell[0] - 1, currCell[1])
        if d == 'S': child = (currCell[0] + 1, currCell[1])
```

**🏁 Start = `(m.rows, m.cols)`** (bottom-right, overridable via `start=`) · **🎯 Goal = `m._goal`**,
default `(1, 1)` (top-left). Search runs bottom-right → top-left, which is why the reconstruction
loops read as reversed.

Each solver returns up to three things — conflating them is the easiest mistake here:

| Return value | Meaning | Animates as |
|---|---|---|
| `bSearch` / `dSeacrh` / `searchPath` | every cell popped from the frontier, in visit order (`list`) | the algorithm *thinking* |
| `bfsPath` / `dfsPath` / `aPath` | the search tree, `{child: parent}` | the frontier tree |
| `fwdPath` | the final route, `{parent: child}` start → goal | the solution walk |

`fwdPath` holds **edges, not cells** — hence `len(fwdPath) + 1` everywhere.
`m.tracePath({agent: path}, delay=ms)` accepts a `dict`, a `list`, or a move string like `'EESN'`
(**not a tuple**); calls queue and play in sequence, and `m.run()` must come last.

🔁 **One generation property is load-bearing.** A recursive-backtracker pass would give a *perfect*
maze (exactly one route between any two cells); this fork then punches openings along the middle row
and column at *p* = 0.9. So **generated mazes always contain cycles** — measured at **41–55 edges
beyond a spanning tree** on six random 20×20 mazes. That is what makes "did it find the *shortest*
route?" a real question; in a perfect maze the comparison would be vacuous.

---

## 🧠 The algorithms

All four search a 4-connected grid with unit step cost (except Dijkstra's hurdles).

- 🌊 **BFS** — `collections.deque`, popped from the left. Equal edge weights mean the first arrival
  at the goal is a shortest route: **always optimal**, at the cost of expanding nearly the whole maze.
- 🕳️ **DFS** — the same loop with a **list used as a stack** (`pop()` instead of `popleft()`). One
  character of difference, completely different behaviour: **no optimality guarantee**, and in a
  looped maze the path runs **1.84× longer than optimal** on average. It also records cells with
  more than one unexplored neighbour in `m.markCells`, drawable via `tracePath(..., showMarked=True)`.
- ⭐ **A\*** — `queue.PriorityQueue` ordered by `f = g + h`. With an admissible `h`, optimal *and*
  cheaper than BFS. Priority is the tuple `(f, h, cell)` — `h` is a deliberate tie-breaker: among
  equal-`f` cells, prefer the one that looks closer.
- 💰 **Dijkstra** — the only solver handling **non-uniform costs**. It takes `agent` objects as
  hurdles, each with a `.cost`, and taxes paths crossing them. With no hurdles it explores the same
  set as BFS and returns the same length, so the demo maze is littered with red cost cells to make
  the difference visible. It picks the next cell by scanning the unvisited set with `min()`, i.e.
  **O(V²)** rather than the textbook O(E log V).

```python
h1 = agent(myMaze, 4, 4, color=COLOR.red); h1.cost = 10
path, total_cost, n_visited = dijkstra(myMaze, h1, h2, h3, h4, h5)
```

| Heuristic | Formula | On a 4-connected grid |
|---|---|---|
| `h_mahattan` | \|dx\| + \|dy\| | ✅ **Exact** — equals true grid distance in an open field; the tightest of the three |
| `h_euclide` | √(dx² + dy²) | Admissible but **loose** — you cannot travel diagonally |
| `h_diagonal` | (dx+dy) + (√2 − 2)·min(dx,dy) | Octile distance, priced for 8-way movement the maze forbids — admissible, loose |

At `(10,10) → (1,1)`: Manhattan **18.00** vs Euclidean **12.73** vs diagonal **12.73**. The latter
two understate the true distance of 18 — which is exactly why they expand ~7% more cells.

---

## 📊 Measured results

30 random **30×30** mazes (900 cells), fixed seed 42, single-threaded on an **AMD EPYC 9534**,
**Python 3.13.12**. Timing is **best-of-5 per maze, median across mazes**. "Expansions" counts
**cells popped from the frontier**, measured identically for every search.

| Algorithm | Path length (mean) | Shortest? | Expansions | % maze searched | Time (ms) |
|---|---:|:---:|---:|---:|---:|
| 🌊 **BFS** | 102.0 | ✅ always | 873 | 97% | 13.43 |
| 🕳️ **DFS** | 179.8 | ❌ 1 / 30 | 342 | 38% | 1.89 |
| ⭐ **A\* (Manhattan)** | 102.0 | ✅ always | 689 | 77% | 2.76 |
| ⭐ **A\* (Euclidean)** | 102.0 | ✅ always | 738 | 82% | 3.20 |
| ⭐ **A\* (diagonal)** | 102.0 | ✅ always | 735 | 82% | 3.37 |
| 💰 **Dijkstra** (no hurdles) | 102.0 | ✅ always | 872 | 97% | 18.13 |

**What the numbers say**

- ✅ **BFS, A\*, and Dijkstra returned identical path lengths on all 30 mazes** — mutual confirmation
  of optimality and a useful cross-check on the implementations.
- ⭐ **A\* gets BFS's guarantee for less search** — 689 vs 873 expansions (−21%) for the same answer.
- 📐 **Heuristic tightness is measurable** — exact Manhattan expands 689 cells; the two loose
  estimates ~737. Looser estimate → more search, exactly as theory predicts.
- 🕳️ **DFS is the cheapest and the worst** — 38% searched in 1.89 ms, but 179.8 cells against an
  optimal 102.0, and shortest in **1 of 30** mazes.
- 💰 **Dijkstra is the slowest while exploring the same cells as BFS** — the cost of its O(V²)
  frontier scan, which is the price of supporting weighted cells.

📓 The notebooks' own `Time` column is a raw `timeit(number=100)` total — **seconds for 100 solves**,
not per-solve. Divided out it agrees with the table above (BFS 1.707 s → ~17 ms/solve).

> The harness that produced this table is **not committed yet** (top of the roadmap below). Treat
> these numbers as reproducible-in-principle — and say so — rather than implying a committed
> benchmark exists.

---


## 👥 Team

Group 6, Introduction to Artificial Intelligence, HUST, 2023.

| Member | ID | Contribution |
|---|---|---|
| **Tan Ngo** | 20210769 | Team lead · maze generation and the Tkinter environment |
| Lê Đức Anh | 20204628 | A\* and the heuristic comparison |
| Nguyễn Cảnh Phước | 20215456 | Depth-first search |
| Mã Tiến Hiệp | 20215365 | Breadth-first search |
| Nguyễn Xuân Hiếu | 20215371 | Dijkstra |

---

