# /loop Command

Plan and launch autonomous loop agents through collaborative discussion.

## Usage

```
/loop                    # Start planning a new loop
/loop status             # Check running loops
/loop attach NAME        # Attach to a session
/loop kill NAME          # Stop a session
```

## Planning Workflow

### Phase 1: Understand Intent

Start by asking what they want to accomplish. One open question:

```yaml
question: "What do you want the loop to work on?"
header: "Goal"
multiSelect: false
options:
  - label: "Build something new"
    description: "New feature, integration, or capability"
  - label: "Improve existing code"
    description: "Refactor, optimize, or clean up"
  - label: "Batch operation"
    description: "Process multiple files, records, or items"
  - label: "Let me explain"
    description: "Describe the work in detail"
```

### Phase 2: Explore and Estimate

**This is the key phase.** Based on their answer:

1. **Explore the codebase** to understand scope:
   - Search for relevant files, patterns, existing implementations
   - Understand the current architecture
   - Identify what needs to change

2. **Estimate complexity** based on findings:
   - Count affected files/components
   - Assess interconnections and dependencies
   - Consider test coverage needed

3. **Propose a plan** with your estimate:
   ```
   Based on exploring the codebase, here's what I found:

   **Scope:** {what needs to change}
   - {file/area 1}: {what changes}
   - {file/area 2}: {what changes}
   - ...

   **My estimate:** {X} iterations ({reasoning})

   **Approach:**
   1. {first major step}
   2. {second major step}
   ...

   Does this match what you had in mind? Should I adjust anything?
   ```

### Phase 3: Iterate Until Aligned

Keep refining until the user confirms:
- Adjust scope up/down based on feedback
- Add/remove components
- Clarify success criteria

Ask:
```yaml
question: "Ready to launch, or want to adjust the plan?"
header: "Confirm"
multiSelect: false
options:
  - label: "Launch it"
    description: "Plan looks good, start the loop"
  - label: "Expand scope"
    description: "Include more in this loop"
  - label: "Reduce scope"
    description: "Focus on less, do the rest later"
  - label: "Let me clarify"
    description: "I want to explain something"
```

### Phase 4: Generate prompt.md

Create `scripts/loop/prompt.md`:

```markdown
# Loop Agent Instructions

## Objective
{clear statement of what to accomplish}

## Context
{what you learned from exploring - key files, patterns, constraints}

## Tasks
{ordered list of specific things to do}
1. {task 1}
2. {task 2}
...

## Per-Iteration Workflow
1. Read progress.txt for context from previous iterations
2. Pick the next incomplete task
3. Implement it fully (code + tests if applicable)
4. Commit with descriptive message
5. Update progress.txt with what was done
6. If all tasks complete: <promise>COMPLETE</promise>

## Success Criteria
{specific, verifiable conditions for "done"}

## Constraints
- One logical change per iteration
- If stuck on same issue for 2 iterations, document and move on
- {any project-specific constraints}
```

### Phase 5: Launch

1. Generate session name from goal: `loop-{slug}`
2. Validate prerequisites exist
3. Start tmux session via tmux-loop skill
4. Update state file
5. Show status:

```
╔════════════════════════════════════════════════════════════╗
║  🚀 Loop Started: loop-{name}                              ║
╠════════════════════════════════════════════════════════════╣
║                                                            ║
║  Estimated: ~{N} iterations                                ║
║  Running with: Opus                                        ║
║                                                            ║
║  Commands:                                                 ║
║    /loop status              - Check all loops             ║
║    /loop attach {name}       - Watch live (Ctrl+b d out)   ║
║    /loop kill {name}         - Stop the loop               ║
║                                                            ║
║  Or directly:                                              ║
║    tmux capture-pane -t loop-{name} -p | tail -30          ║
║                                                            ║
╚════════════════════════════════════════════════════════════╝
```

## Subcommands

### /loop status
Show all running loop sessions with age and status.

### /loop attach NAME
Attach to watch live. Remind: `Ctrl+b` then `d` to detach.

### /loop kill NAME
Confirm, then terminate session and clean up state.

## Key Principle

**Claude estimates, user confirms.** Don't ask the user how complex something is - explore the codebase and tell them what you found. They refine from there.
