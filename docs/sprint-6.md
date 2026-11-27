# Phase 2 — Sprint 6

November 19, 2026. Status: **COMPLETE**.

## Available Features

**Algorithms Lab** opens from the existing menu at `#/algorithms-lab`. The page is lazy-loaded and uses CardiGuard's colors, fonts, theme, and responsive layout.

| **Category** | **Options** |
|---|---|
| Data structures | Array, Linked List, Stack, Queue, Circular Queue, Hash Table, Binary Tree, Graph, Priority Queue |
| Sorting | Bubble Sort, Merge Sort |
| Searching | Binary Search |
| Graphs | BFS, DFS, Dijkstra |
| Window algorithms | Sliding Window for maximum sum with a fixed window size |

The Python implementations are custom-built. The Array shifts elements by index; Linked List, Stack, and Queue use their own nodes and links. Circular Queue uses modulo arithmetic. Hash Table uses deterministic hashing and separate chaining, Binary Tree is an unbalanced BST, and Priority Queue is a stable min-heap. Built-in `heapq`, `deque`, and ready-made sorting/graph algorithms are not used.

Each option includes a working example, editable input, explanation, input/output, pseudocode, time/space complexity, advantages, limitations, and practical applications. Data structure operations can be selected and added to a queue; results and errors are preserved. BST supports insert/find/remove and inorder/preorder/postorder traversal.

## Execution and Visualization

The backend executes the algorithm on the input and returns independent state snapshots for each execution step. The frontend displays these steps. Sorting visualizes comparisons/swaps/writes, searching visualizes intervals/midpoints, and graphs display node, edge, weight, current/visited/frontier, and Dijkstra predecessor/distance states. Separate visualizations are available for Stack, Queue, Linked List, Circular Buffer, Hash Table buckets, Tree, and Heap.

- **Run** executes the new input; for the same input, it resumes paused playback.
- **Pause** stops at the current step.
- **Reset** returns to the first step and stops playback.
- **Step Forward** advances a paused execution by one step.
- **0.5x / 1x / 2x / 4x** set the delay between steps to 1600/800/400/200 ms, respectively.

The current step, status, and corresponding pseudocode line are displayed. The execution log shows messages up to the current step. At the end, returned values and measured operation counts are displayed. Input changes are applied on the next Run. When the algorithm changes, the previous request and playback are cancelled. The existing 15-second timeout/retry hook is used for requests.

**Compare** executes the same input using Bubble Sort and Merge Sort. Two actual execution traces are displayed side by side; the shorter trace remains at its final step. The number of comparisons, swaps, and array writes is returned. These are educational counters, not wall-clock benchmarks. `operations` counts adjacency entries examined for graphs, supplied operations for data structures, and removal/addition steps for Sliding Window. These counters do not represent the total number of Python instructions.

## Backend API

| **Endpoint** | **Response** |
|---|---|
| `GET /api/v1/algorithms` | 16 options, metadata, operation fields, and example requests |
| `POST /api/v1/algorithms/execute` | `runs[]`, `step_count`; each run contains `steps`, `result`, and `metrics` |

Example request:

```json
{
  "algorithm": "bubble-sort",
  "values": [3, 2, 1],
  "compare": true
}
```

Each step contains `line`, `message`, typed `state`, active/completed indices, range, current/visited/frontier, distances, path_edges, and error fields. The API contract is also available locally in `/openapi.json`. The new endpoints have been added alongside the previous 7 ECG/AI/health endpoints. No dataset or ML model is required for this laboratory.

## Validation and Limits

- A maximum of 24 integers in the initial array, each ranging from −999 to 999; a maximum of 24 data structure operations.
- Graphs support a maximum of 12 unique nodes and 32 edges; node names must contain 1–12 ASCII letters, digits, `_`, or `-`.
- Edge weights and priorities must be finite numbers between −999 and 999. Dijkstra does not accept negative edge weights.
- Circular Queue and Hash Table capacities range from 1 to 16; hash key length ranges from 1 to 20.
- Window width ranges from 1 to 24 and must fit within a non-empty array. Binary Search requires sorted input.
- A defensive limit of 4096 trace steps applies to each run. Input limits keep execution local and short.

Invalid formats, unknown operations, missing fields, and incompatible initial states return 422. Operation errors such as removing from an empty structure, adding to a full buffer, or using a non-existent index/key are returned through `steps[].error=true` and `result.errors`. The previous state is preserved, and subsequent operations continue. The UI reports these errors separately at the end. Execution only calls permitted operation names; users cannot submit Python or JavaScript code.

Empty arrays and graphs are supported. If Binary Search does not find a value, `index=null`; for an empty Sliding Window, `max_sum=null`; for unreachable nodes in Dijkstra, the distance is `null` and the path is `[]`. Traversal visits the portion of the graph reachable from the start node. The input order of nodes and edges makes tie-breaking deterministic. The min-heap preserves insertion order when priorities are equal.

## Created Files

| **File** | **Purpose** |
|---|---|
| `backend/app/algorithms/__init__.py` | Module package |
| `backend/app/algorithms/structures.py` | 9 data structures |
| `backend/app/algorithms/execution.py` | 7 algorithms, operations, and trace generation |
| `backend/app/algorithms/catalog.py` | Educational content and examples |
| `backend/app/algorithms/schemas.py` | Pydantic request/response models |
| `backend/app/algorithms/routes.py` | 2 endpoints |
| `backend/tests/test_algorithms.py` | 61 data structure/algorithm/API/validation tests |
| `frontend/src/lib/algorithms.ts` | TypeScript types |
| `frontend/src/features/algorithms/AlgorithmsLab.tsx` | Page, forms, and shared execution controls |
| `frontend/src/features/algorithms/AlgorithmVisualization.tsx` | State-based visualizations |
| `frontend/src/features/algorithms/algorithms.css` | Responsive styling and transitions |
| `frontend/src/features/algorithms/AlgorithmsLab.test.tsx` | Controls, speed, input, retry, and navigation tests |
| `frontend/e2e/algorithms.spec.ts` | 4 scenarios × 3 screen sizes using the real API |
| `docs/sprint-6.md` | This report |

Modified files: `backend/app/main.py` connects the router; `frontend/src/App.tsx` opens the lazy-loaded page; `frontend/src/features/overview/Overview.tsx` displays the Available status. `frontend/e2e/heart.spec.ts` was adjusted to match the existing 3D page's `.heart-structure-list` selector. `README.md` and `docs/development.md` update the usage and code guides. No new runtime or development dependencies were added.

## Actual Verification Results

| **Command** | **Result** |
|---|---|
| `backend/` — `.venv/bin/python -m pytest` | **215 passed** |
| `frontend/` — `npm test` | **54 passed**, 12 test files |
| `frontend/` — `npm run build` | TypeScript and production build passed |
| `frontend/` — `npm run test:e2e` | **51 passed**, desktop/tablet/mobile |
| Project root — `git diff --check` | Passed |

Backend tests cover boundary/invalid inputs, collisions, wraparound, empty/full structures, BST removal, heap ties, random sorting, graph cycles/unreachable nodes/zero-weight edges/stale heap entries, determinism, all 16 catalog examples, and the API. Frontend tests verify controls, speed, pseudocode, forms, and error recovery using actual returned frames.

Browser verification was performed in Chromium at desktop 1440×1000, tablet 1024×1366, and mobile 393×852 resolutions. ECG/AI/3D regression scenarios are also included in the same test suite. Screenshot/report artifacts are stored in `.cache/browser-tests` and `.cache/browser-report`. No separate lint command is configured; the build also performs TypeScript checks.

A total of **320 tests passed**, with no skipped tests. Of these, 76 were added in Sprint 6: 61 backend tests, 3 frontend tests, and 12 browser tests. All 16 new options were executed using the actual local API. Sorting comparison, controls, graph distance/path, input errors, and empty inputs were tested. Mobile graph and desktop light/dark screenshots were reviewed; horizontal overflow was checked at all three screen sizes. All previous ECG, AI, and 3D heart browser scenarios passed.

In the final production build, the Algorithms Lab JavaScript chunk is **19.27 KB**, or **6.10 KB** when gzipped. There are no TypeScript errors.

## Running Locally

From the project root, open two terminals.

Terminal 1:

```bash
cd backend
.venv/bin/python -m uvicorn app.main:app --reload --host 127.0.0.1 --port 8000
```

Terminal 2:

```bash
cd frontend
npm run dev
```

Open `http://127.0.0.1:5173/#/algorithms-lab` in your browser. The existing dependencies are sufficient. Browser tests start their own servers on ports 18000/15173.

## Limitations and Decisions

- Comparison is available for Bubble Sort and Merge Sort. Both use the same input and the same playback step.
- Binary Tree is an unbalanced BST; separate AVL/red-black balancing implementations are not included.
- A graph stores one edge per node pair. The weight of an existing edge can be updated through an operation.
- Sliding Window is a fixed-width example for finding the maximum sum. When multiple windows have the same maximum sum, the first window is selected.
- The trace is calculated in full before being displayed locally. Streaming and large-dataset benchmarking are not supported.
- Big-O describes the work performed by the main algorithm. The sorted-input validation for Binary Search takes O(n). Copying snapshots and JSON transfer require additional time and memory. Dijkstra displays O((V + E) log(V + E)) time and O(V + E) additional space for the lazy heap implementation.
- The Vite large-chunk warning relates to the existing Plotly and Three.js components. Algorithms Lab is a separate lazy-loaded chunk. Browser verification is limited to Chromium.
- Examples are intended for educational purposes; they do not provide clinical diagnoses or medical decisions.

Technical references for complexity descriptions: [NIST Binary Search](https://xlinux.nist.gov/dads/HTML/binarySearch.html), [MIT 6.006 Lecture Notes](https://ocw.mit.edu/courses/6-006-introduction-to-algorithms-spring-2020/resources/lecture-notes/).

## Completion Decision

**Sprint 6 COMPLETE.** The required 9 data structures and 7 algorithms, real execution traces, custom input, playback controls, comparison, educational content, API, and responsive UI are ready. Backend, frontend, production build, and browser checks covering previous functionality have passed. There are no outstanding blockers for the required features.
