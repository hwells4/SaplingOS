# Loop

Autonomous execution of multi-step plans with context management.

## Before you start

Generate your plan and tasks first:

```bash
# 1. Define what you're building
/generate-prd

# 2. Break it into executable tasks
/generate-stories
```

This creates `prd.json` with tasks the loop can execute autonomously.

> **Note:** Currently optimized for coding tasks (commits, tests, verification). Could be generalized for other work types.

## When to use

- Extending Sapling OS with new features
- Long-running batch processing (e.g., process 1000 documents)
- Any multi-step work that benefits from fresh context per task
- Plans too large to hold in a single conversation

## How it works

```
/generate-prd → /generate-stories → prd.json → loop.sh → Autonomous execution
```

1. **Plan**: `/generate-prd` defines what you're building
2. **Tasks**: `/generate-stories` breaks it into executable tasks with acceptance criteria
3. **Configure**: Create `prompt.md` with instructions for how the agent should work
4. **Run**: `loop.sh` picks a task, implements it, commits, repeats
5. **Learn**: `progress.txt` accumulates patterns across iterations

## Files

| File | Purpose |
|------|---------|
| `prd.json` | Tasks with acceptance criteria (from /generate-stories) |
| `progress.txt` | Accumulated learnings across iterations |
| `prompt.md` | Instructions for each iteration (you create this) |

## Usage

```bash
# Test single iteration first
./loop-once.sh

# Run autonomously (default 25 iterations)
./loop.sh

# Run with custom limit
./loop.sh 50

# Archive completed work and start fresh
./loop-archive.sh "feature-name"
```

## The loop

Each iteration:
1. Agent reads `prompt.md` + `prd.json` + `progress.txt`
2. Picks next incomplete task
3. Implements and verifies
4. Commits changes
5. Updates `progress.txt` with learnings
6. Signals `<promise>COMPLETE</promise>` when all done

Fresh context each iteration prevents degradation on long runs.
