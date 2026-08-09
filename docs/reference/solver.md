# Solver

OM Core includes a built-in optimization solver that lets you find optimal values for decision variables in your model, subject to constraints. The solver evaluates your model's rules live during optimization—no export or model duplication is required.

## Overview

The solver works by:

1. **Selecting decision variables** — cells in your cubes that the optimizer is allowed to change.
2. **Defining objectives** — cells whose values should be minimized or maximized.
3. **Specifying constraints** — bounds on other cells that must be satisfied.
4. **Running an optimization algorithm** — the solver iteratively proposes candidate values, the engine evaluates the model, and the algorithm converges toward an optimal solution.
5. **Applying results** — optimized values are written back to the workspace as hard values.

The solver runs asynchronously as a background job. You start a job, poll its status, retrieve the result, and optionally apply it to the workspace.

## Key terms

- **Decision variable** — a cell whose value the solver is free to adjust. Each variable has optional lower and upper bounds.
- **Objective** — a cell whose value the solver tries to minimize or maximize. A problem can have one objective (single-objective) or multiple objectives (multi-objective).
- **Constraint** — a restriction on a cell's value. Supported types: `lower` (cell >= bound), `upper` (cell <= bound), `range` (lower <= cell <= upper), and `equality` (cell == bound).
- **Backend** — the optimization library that runs the algorithm. OM Core ships with two backends: `scipy` (for single-objective) and `pymoo` (for multi-objective).
- **Algorithm** — the specific optimization method used by a backend (e.g. COBYLA, SLSQP, NSGA-II).
- **Job** — an asynchronous solver run. Each job has a lifecycle: `running` → `cancelling` → `finished`.
- **Termination status** — how the job ended: `optimal`, `feasible`, `infeasible`, `cancelled`, `timeout`, `failed`, or `limit_exceeded`.
- **Pareto front** — the set of non-dominated solutions produced by a multi-objective optimizer. Each solution represents a trade-off between competing objectives.
- **Apply** — writing optimized variable values back to the workspace as hard values, creating a new revision.
- **Cell reference** — the address of a cell in `Cube:Dim.Item` or `Cube::Dim.Item:Dim.Item` format, used to identify variables, objectives, and constraints.

## Usage

### Building a problem spec interactively

The REPL provides a set of `solver` commands for building and running optimization problems:

```
om> solver new
om> solver variable add PF::Stock.S01:Metric.weight 0 100
om> solver variable add PF::Stock.S02:Metric.weight 0 100
om> solver objective add PF::Output.S01:Metric.profit max
om> solver constraint add PF::Output.S01:Metric.cost upper 5000
om> solver limit set max_iterations 500
om> solver run
```

This creates a draft problem spec with two decision variables (bounded between 0 and 100), one objective (maximize profit), one constraint (cost <= 5000), a 500-iteration limit, and starts the solver job.

### Polling status and retrieving results

```
om> solver status <job_id>
om> solver result <job_id>
```

`solver status` returns the current job state and evaluation count. `solver result` returns the solution (variable values, objective values) once the job is finished.

### Applying results

```
om> solver apply <job_id>
```

For single-objective results, this writes the optimized variable values back to the workspace. The apply is atomic—it opens an engine transaction, revalidates the base revision, writes values, recalculates, and commits.

For multi-objective results, specify a Pareto index:

```
om> solver apply <job_id> 2
```

### Running inline JSON (advanced)

You can pass a complete problem spec as JSON:

```json
{
  "backend": "scipy",
  "algorithm": "cobyla",
  "variables": [
    {"ref": "PF::Stock.S01:Metric.weight", "lower": 0, "upper": 100},
    {"ref": "PF::Stock.S02:Metric.weight", "lower": 0, "upper": 100}
  ],
  "objectives": [
    {"ref": "PF::Output.S01:Metric.profit", "direction": "max"}
  ],
  "constraints": [
    {"ref": "PF::Output.S01:Metric.cost", "type": "upper", "bound": 5000}
  ],
  "limits": {"max_iterations": 500, "tol": 1e-6}
}
```

```
om> solver run '{"backend":"scipy","algorithm":"cobyla",...}'
```

### Waiting for completion

Add `--wait` to `solver run` to block until the job finishes, with progress updates:

```
om> solver run --wait
```

### Cancelling a job

```
om> solver cancel <job_id>
```

Cancellation is cooperative—the solver checks the cancellation token between iterations and stops gracefully.

### Exporting a diagnostic report

```
om> solver export <job_id> [file_path]
```

Exports a JSON report with job metadata, telemetry, termination status, variable/objective values, and apply state.

### Dumping results to a cube

After a solver run, you can dump the results into a cube for inspection and comparison:

```
om> solver dump [job_id] [--cube <name>]
```

If `job_id` is omitted, the most recent finished job is used. If `--cube` is omitted, the cube defaults to `RESULT`.

This creates (or reuses) a cube with three dimensions—`ResultTag`, `ResultPoint`, and `ResultMetric`—and writes each solution point's variable and objective values as hard values. A view is automatically created with `ResultMetric` on rows, `ResultPoint` on columns, and `ResultTag` on the page axis.

## Backends and algorithms

### SciPy backend (`scipy`)

The default backend, suited for single-objective nonlinear optimization. Uses [SciPy's](https://docs.scipy.org/doc/scipy/reference/optimize.html) optimization routines.

| Algorithm | Description | Bounds | Inequality constraints | Equality constraints | Derivatives |
|---|---|:---:|:---:|:---:|:---:|
| `auto` | Automatic selection based on problem structure | Yes | Yes | Yes | No |
| `cobyla` | Derivative-free, handles all constraint types | Yes | Yes | Yes | No |
| `slsqp` | Gradient-based, good for smooth constrained problems | Yes | Yes | Yes | No |
| `trust-constr` | Robust for large-scale constrained problems | Yes | Yes | Yes | No |
| `nelder-mead` | Simplex-based, unconstrained or bounds only | Yes | No | No | No |
| `powell` | Conjugate direction, unconstrained or bounds only | Yes | No | No | No |
| `linprog` | Linear programming (HiGHS solver) | Yes | Yes | Yes | No |
| `bobyqa` | Derivative-free, bounds only | Yes | No | No | No |
| `newuoa` | Derivative-free, unconstrained | No | No | No | No |
| `lincoa` | Derivative-free, linear constraints | Yes | Yes | No | No |

### pymoo backend (`pymoo`)

Suited for multi-objective optimization. Uses [pymoo's](https://pymoo.org/) evolutionary algorithms.

| Algorithm | Description | Best for |
|---|---|---|
| `auto` | Automatic selection based on objective count | General use |
| `nsga2` | NSGA-II | 2–3 objectives |
| `nsga3` | NSGA-III | Many objectives (4+) |
| `moead` | Decomposition-based | Multi-objective with constraints |
| `sms-emoa` | Hypervolume-based selection | 2 objectives |
| `ga` | Single-objective genetic algorithm | Single-objective with pymoo |

To use the pymoo backend, set it in the problem spec:

```
om> solver option set backend pymoo
om> solver algorithm nsga2
```

## Single-objective optimization

Single-objective optimization finds the best value for one objective cell. The solver proposes candidate values for the decision variables, the engine evaluates the model, and the algorithm iterates until convergence.

### Example: maximize profit

```
om> solver new
om> solver variable add PF::Stock.S01:Metric.weight 0 100
om> solver variable add PF::Stock.S02:Metric.weight 0 100
om> solver objective add PF::Output.S01:Metric.profit max
om> solver constraint add PF::Output.S01:Metric.cost upper 5000
om> solver algorithm cobyla
om> solver run --wait
```

After the job finishes, the result contains:

- **Variable values** — the optimal values for each decision variable.
- **Objective values** — the achieved objective value.
- **Constraint values** — the constraint cell values at the solution (to verify feasibility).
- **Termination status** — `optimal` if the solver converged, `feasible` if a feasible point was found but not proven optimal.

Apply the result to write the optimized values back to the workspace:

```
om> solver apply <job_id>
```

### Choosing an algorithm

- **COBYLA** (default) — derivative-free, handles all constraint types. Good general-purpose choice.
- **SLSQP** — efficient for smooth problems with constraints. May converge faster than COBYLA on well-behaved models.
- **trust-constr** — robust for large-scale or poorly conditioned problems.
- **Nelder-Mead / Powell** — unconstrained or bounds-only problems. Simpler but less reliable for constrained models.
- **linprog** — linear programming. Requires the problem to be linear (use `problem_class: linear` in the spec).

## Multi-objective optimization

Multi-objective optimization finds a set of Pareto-optimal solutions—solutions where no objective can be improved without worsening another. This is useful when you have competing goals (e.g. maximize profit while minimizing risk).

### Example: maximize profit and minimize risk

```
om> solver new
om> solver option set backend pymoo
om> solver algorithm nsga2
om> solver variable add PF::Stock.S01:Metric.weight 0 100
om> solver variable add PF::Stock.S02:Metric.weight 0 100
om> solver objective add PF::Output.S01:Metric.profit max
om> solver objective add PF::Output.S01:Metric.risk min
om> solver run --wait
```

After the job finishes, the result contains a **Pareto front**—a list of non-dominated solutions, each with its own variable values and objective values.

### Selecting and applying a Pareto solution

List the Pareto front:

```
om> solver result <job_id>
```

Output shows each solution with its index and objective values:

```
  Pareto front: 15 solutions
    [0]  objectives=[0.85, 0.12]
    [1]  objectives=[0.78, 0.08]
    ...
```

Apply the solution that best fits your trade-off:

```
om> solver apply <job_id> 1
```

### Choosing a multi-objective algorithm

- **NSGA-II** — the most popular multi-objective evolutionary algorithm. Good default for 2–3 objectives.
- **NSGA-III** — designed for many-objective problems (4+ objectives). Uses reference directions to maintain diversity.
- **MOEAD** — decomposition-based approach. Effective for problems with complex Pareto fronts.
- **SMS-EMOA** — hypervolume-based selection. Good for 2-objective problems where hypervolume is a meaningful indicator.

### Population size and iterations

Multi-objective algorithms use a population of candidate solutions. Control this with limits:

```
om> solver limit set pop_size 200
om> solver limit set max_iterations 300
```

The total number of evaluations is approximately `pop_size * max_iterations`.

## Limits and stopping criteria

| Limit | Description | Default |
|---|---|---|
| `max_iterations` | Maximum number of iterations (or generations for evolutionary algorithms) | 1000 |
| `tol` | Tolerance for convergence | 1e-6 |
| `max_wall_time_seconds` | Wall-clock timeout in seconds | 300 |
| `pop_size` | Population size (pymoo only) | 100 |
| `seed` | Random seed for reproducibility (pymoo only) | 42 |

Limits are validated against the runtime policy, which enforces upper bounds on iterations, wall time, and concurrency.

## Job lifecycle

```
running ──→ finished
   │
   └──→ cancelling ──→ finished
```

1. **`running`** — the solver is actively evaluating candidates.
2. **`cancelling`** — cancellation has been requested; the solver will stop at the next check point.
3. **`finished`** — the job has ended. Check `termination_status` for the outcome.

### Termination statuses

| Status | Meaning |
|---|---|
| `optimal` | The solver found an optimal solution. |
| `feasible` | A feasible solution was found, but optimality was not proven. |
| `infeasible` | No feasible solution exists for the given constraints. |
| `cancelled` | The job was cancelled by the user. |
| `timeout` | The wall-clock time limit was reached. |
| `failed` | The solver encountered an error. |
| `limit_exceeded` | A policy limit (e.g. max evaluations) was exceeded. |

Only results with status `optimal` or `feasible` can be applied (unless `allow_nonoptimal` is set in the policy).

## REPL command reference

| Command | Description |
|---|---|
| `solver new` | Reset the draft problem spec to defaults. |
| `solver show` | Display the current draft problem spec. |
| `solver variable add <ref> [lower] [upper]` | Add a decision variable with optional bounds. |
| `solver variable remove <idx>` | Remove a variable by index. |
| `solver variable list` | List all variables. |
| `solver objective add <ref> <min\|max>` | Add an objective. |
| `solver objective remove <idx>` | Remove an objective by index. |
| `solver objective list` | List all objectives. |
| `solver constraint add <ref> <type> <bound> [bound2]` | Add a constraint (`range` type takes two bounds). |
| `solver constraint remove <idx>` | Remove a constraint by index. |
| `solver constraint list` | List all constraints. |
| `solver algorithm <id>` | Set the optimization algorithm. |
| `solver option set <key> <value>` | Set a solver option (e.g. `backend`). |
| `solver option list` | Show current options. |
| `solver option clear` | Clear all options. |
| `solver limit set <key> <value>` | Set a stopping criterion. |
| `solver limit list` | Show current limits. |
| `solver run [json_spec] [--wait]` | Start a solver job. |
| `solver status [job_id]` | Poll job status (lists all jobs if no ID given). |
| `solver result <job_id>` | Retrieve the result of a finished job. |
| `solver apply [job_id] [index\|apply_request_id]` | Apply results to the workspace (auto-applies most recent finished job if no args). |
| `solver cancel <job_id>` | Cancel a running job. |
| `solver backends` | List registered backends. |
| `solver algorithms [backend_id]` | List algorithms for a backend. |
| `solver export <job_id> [file_path]` | Export a diagnostic report. |
| `solver dump [job_id] [--cube <name>]` | Dump results into a cube (uses most recent finished job and cube `RESULT` by default). |

## Cell reference format

Solver commands use cell references to identify variables, objectives, and constraints. The format is:

```
Cube::Dim.Item
Cube::Dim.Item:Dim.Item     (multi-dimension)
```

For example:

- `PF::Stock.S01` — cell in cube `PF`, dimension `Stock`, item `S01`.
- `PF::Stock.S01:Metric.weight` — cell in cube `PF`, dimensions `Stock` and `Metric`, items `S01` and `weight`.
