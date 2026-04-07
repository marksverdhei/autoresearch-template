# autoresearch

This is an experiment to have an LLM agent autonomously optimize a measurable metric through iterative experimentation.

## Project context

<!-- [FILL IN] Describe what you're optimizing and why. Example:
"We are optimizing a sorting algorithm for throughput on arrays of 10M integers."
"We are optimizing an LLM training script to achieve the lowest validation loss in a fixed time budget."
"We are optimizing prompt templates to maximize an LLM-judge quality score."
-->

## Metric

<!-- [FILL IN] Define the single metric being optimized. -->

- **Name**: `score`
- **Direction**: lower is better <!-- or: higher is better -->
- **How to extract**: `grep '^score:' run.log | awk '{print $2}'`

<!-- The metric MUST be a single number that can be extracted programmatically from the experiment output. -->

## Setup

To set up a new experiment, work with the user to:

1. **Agree on a run tag**: propose a tag based on today's date (e.g. `apr7`). The branch `autoresearch/<tag>` must not already exist.
2. **Create the branch**: `git checkout -b autoresearch/<tag>` from current master.
3. **Read the in-scope files**: Read all relevant files for full context.
   <!-- [FILL IN] List the files the agent should read. Example:
   - `README.md` — repository context
   - `evaluate.py` — fixed evaluation harness (do not modify)
   - `solution.py` — the file you modify
   -->
4. **Initialize results.tsv**: Create `results.tsv` with just the header row. The baseline will be recorded after the first run.
5. **Confirm and go**: Confirm setup looks good.

Once you get confirmation, kick off the experimentation.

## Experimentation

<!-- [FILL IN] Describe how to run a single experiment. Example:
"Run `uv run solution.py > run.log 2>&1` to execute the benchmark."
"Run `./run.sh > run.log 2>&1` to execute the experiment."
-->

**Run command**: `[FILL IN] > run.log 2>&1`

**What you CAN do:**
<!-- [FILL IN] List the files the agent may modify and what kinds of changes are allowed. Example:
- Modify `solution.py` — algorithm, data structures, parameters, everything is fair game.
-->

**What you CANNOT do:**
<!-- [FILL IN] List constraints. Example:
- Modify the evaluation harness or benchmark
- Install new packages or add dependencies
- Modify any file other than solution.py
-->

**The goal is simple: get the best score.** Everything within the allowed files is fair game: architecture, parameters, algorithms, implementation details. The only constraint is that the code runs without crashing and produces a valid metric.

**Simplicity criterion**: All else being equal, simpler is better. A small improvement that adds ugly complexity is not worth it. Removing code and getting equal or better results is a great outcome.

## Output format

<!-- [FILL IN] Describe what the run produces and how to extract the metric. Example: -->

Once the experiment finishes it prints a summary like this:

```
---
score:    0.9979
```

Extract the key metric:

```
grep "^score:" run.log
```

## Logging results

When an experiment is done, log it to `results.tsv` (tab-separated, NOT comma-separated).

The TSV has a header row. The minimum columns are:

```
commit	score	status	description
```

1. git commit hash (short, 7 chars)
2. score achieved — use 0.0 for crashes
3. status: `keep`, `discard`, or `crash`
4. short text description of what this experiment tried

<!-- [OPTIONAL] Add extra columns for your domain. Example:
```
commit	score	memory_gb	status	description
```
-->

## The experiment loop

The experiment runs on a dedicated branch (e.g. `autoresearch/apr7`).

LOOP FOREVER:

1. Look at the git state: the current branch/commit we're on
2. Modify the target file(s) with an experimental idea
3. git commit
4. Run the experiment: `[RUN COMMAND] > run.log 2>&1` (redirect everything — do NOT let output flood your context)
5. Extract the results: `grep "^score:" run.log`
6. If the grep output is empty, the run crashed. Run `tail -n 50 run.log` to read the error and attempt a fix. If you can't get things to work after a few attempts, give up on this idea.
7. Record the results in the TSV (NOTE: do not commit the results.tsv file, leave it untracked by git)
8. If score improved (better in the configured direction), you "advance" the branch, keeping the git commit
9. If score is equal or worse, you `git reset` back to where you started

**Timeout**: If a run exceeds 2x the expected duration, kill it and treat it as a failure.

**Crashes**: If a run crashes, use your judgment: fix trivial bugs and re-run. If the idea is fundamentally broken, log "crash" and move on.

**NEVER STOP**: Once the experiment loop has begun, do NOT pause to ask the human if you should continue. The human might be asleep or away. You are autonomous. If you run out of ideas, think harder — re-read the files for new angles, try combining previous near-misses, try more radical changes. The loop runs until the human interrupts you.

As a rough guide: if each experiment takes ~5 minutes, you can run ~12/hour, ~100 overnight. The user wakes up to a full experiment log and a progress plot.
