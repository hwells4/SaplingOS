# /loop Command

Plan and launch autonomous loop agents in tmux sessions.

## What This Command Does

1. **Plan** - Help you define what the loop should accomplish
2. **Generate** - Create the prompt.md and task structure
3. **Launch** - Start the loop in a tmux session
4. **Monitor** - Show you how to check on progress

## Usage

```
/loop                    # Start planning a new loop
/loop status             # Check running loops
/loop attach NAME        # Attach to a session
/loop kill NAME          # Stop a session
```

## Workflow

### Step 1: Understand the Goal

Ask the user:
```yaml
question: "What do you want the loop to accomplish?"
header: "Goal"
multiSelect: false
options:
  - label: "Build a feature"
    description: "Implement something new with multiple steps"
  - label: "Batch processing"
    description: "Process many items (files, records, etc.)"
  - label: "Refactoring"
    description: "Improve existing code across multiple files"
  - label: "Something else"
    description: "Describe via Other"
```

### Step 2: Scope the Work

Based on their goal, ask:
```yaml
question: "How complex is this work?"
header: "Scope"
multiSelect: false
options:
  - label: "Small (5-10 iterations)"
    description: "Quick task, should finish in under an hour"
  - label: "Medium (15-25 iterations)"
    description: "Moderate work, 1-2 hours"
  - label: "Large (30-50 iterations)"
    description: "Substantial work, may take several hours"
  - label: "Custom"
    description: "Specify iteration count"
```

### Step 3: Generate the Plan

Create `scripts/loop/prompt.md` with:

```markdown
# Loop Agent Instructions

## Objective
{user's goal}

## Scope
- Max iterations: {count}
- Focus: {specific focus areas}

## Per-Iteration Workflow
1. Check progress.txt for context from previous iterations
2. Identify next task to complete
3. Implement and verify
4. Commit changes with descriptive message
5. Update progress.txt with what was done and learnings
6. If all work complete, output: <promise>COMPLETE</promise>

## Success Criteria
{what "done" looks like}

## Constraints
- One logical change per iteration
- Always run tests before committing
- If stuck for 2+ iterations on same issue, document blocker and move on
```

### Step 4: Create Session Name

Generate a name from their goal:
- `loop-{feature-slug}` (e.g., `loop-auth-system`, `loop-batch-migrate`)

### Step 5: Launch

Use the tmux-loop skill to:
1. Validate prerequisites (loop.sh, prompt.md exist)
2. Start the tmux session
3. Update state file
4. Show monitoring instructions

### Step 6: Handoff

Display:
```
╔══════════════════════════════════════════════════════════════╗
║  🚀 Loop Started: loop-{name}                                ║
╠══════════════════════════════════════════════════════════════╣
║                                                              ║
║  The loop is now running autonomously in the background.     ║
║                                                              ║
║  Quick commands:                                             ║
║    Peek:    tmux capture-pane -t loop-{name} -p | tail -30   ║
║    Attach:  tmux attach -t loop-{name}                       ║
║    Status:  /loop status                                     ║
║    Stop:    /loop kill loop-{name}                           ║
║                                                              ║
║  The loop will signal <promise>COMPLETE</promise> when done. ║
║  I'll warn you about stale sessions (>2 hours).              ║
║                                                              ║
╚══════════════════════════════════════════════════════════════╝
```

## Subcommands

### /loop status
Run: `bash .claude/skills/tmux-loop/scripts/check-sessions.sh`

### /loop attach NAME
Follow: `.claude/skills/tmux-loop/workflows/attach-session.md`

### /loop kill NAME
Follow: `.claude/skills/tmux-loop/workflows/kill-session.md`

### /loop cleanup
Follow: `.claude/skills/tmux-loop/workflows/cleanup-sessions.md`

## Files Created/Modified

- `scripts/loop/prompt.md` - Instructions for the loop agent
- `.claude/loop-sessions.json` - Session tracking state
- `scripts/loop/progress.txt` - Accumulates learnings (created by loop)

## Integration

This command uses:
- `tmux-loop` skill for session management
- `scripts/loop/loop.sh` for execution (runs with `--model opus`)
