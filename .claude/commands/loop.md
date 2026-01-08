# /loop Command

Plan, generate tasks, and launch autonomous loop agents.

## Usage

```
/loop                    # Full flow: plan → tasks → launch
/loop status             # Check running loops
/loop attach NAME        # Attach to a session
/loop kill NAME          # Stop a session
```

## Full Planning Flow

### Phase 1: Understand Intent

Ask what they want to accomplish:

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
  - label: "I have a PRD ready"
    description: "Skip to task generation"
```

### Phase 2: Research with Subagents

**Spawn subagents to gather context in parallel:**

```
Task(subagent_type="Explore", prompt="Find all files related to {topic}. Understand current architecture and patterns.")

Task(subagent_type="Explore", prompt="Search for existing implementations of {similar feature}. Note patterns we should follow.")
```

This gives you real codebase knowledge to inform the plan.

### Phase 3: Generate PRD (if needed)

If no PRD exists, invoke the generate-prd skill:

```
Skill(skill="generate-prd")
```

This asks adaptive questions until confident, then creates:
```
brain/outputs/{date}-{slug}-prd.md
```

### Phase 4: Generate Stories → prd.json

Invoke generate-stories skill:

```
Skill(skill="generate-stories")
```

This:
1. Reads the PRD
2. Breaks into small, executable stories
3. Generates acceptance criteria with test cases
4. Creates `scripts/loop/prd.json`

### Phase 5: Ensure prompt.md Exists

If `scripts/loop/prompt.md` doesn't exist, create the standard template:

```markdown
# Loop Agent Instructions

You are an autonomous agent executing tasks from prd.json.

## Per-Iteration Workflow

1. **Read context:**
   - `prd.json` - Find next story where `passes: false`
   - `progress.txt` - Learn from previous iterations

2. **Execute the story:**
   - Implement the feature/fix
   - Verify ALL acceptance criteria pass
   - Run test commands (npm test, typecheck, etc.)

3. **Commit:**
   - Stage relevant changes only
   - Write descriptive commit message referencing story ID
   - Example: "US-003: Add email validation with test cases"

4. **Update progress:**
   - Append to progress.txt what you did and learned
   - Note any blockers or patterns discovered

5. **Mark complete:**
   - Update prd.json: set `passes: true` for the story
   - Add any notes about implementation

6. **Check if done:**
   - If ALL stories have `passes: true`: output `<promise>COMPLETE</promise>`
   - Otherwise: end iteration (next iteration picks up next story)

## Constraints

- One story per iteration (fresh context each time)
- If stuck on same story for 2 iterations, document blocker and skip
- Always run verification commands before marking complete
- Never mark a story complete if tests fail

## Files

| File | Purpose |
|------|---------|
| prd.json | Stories with acceptance criteria |
| progress.txt | Accumulated learnings |
| prompt.md | These instructions (you're reading it) |
```

### Phase 6: Launch in tmux

1. Generate session name: `loop-{feature-slug}`
2. Start session:
   ```bash
   tmux new-session -d -s "loop-NAME" -c "$(pwd)" './scripts/loop/loop.sh'
   ```
3. Update `.claude/loop-sessions.json`
4. Display:

```
╔════════════════════════════════════════════════════════════╗
║  🚀 Loop Launched: loop-{name}                             ║
╠════════════════════════════════════════════════════════════╣
║                                                            ║
║  Stories: {N} tasks in prd.json                            ║
║  Model: Opus                                               ║
║                                                            ║
║  Commands:                                                 ║
║    /loop status          - Check all loops                 ║
║    /loop attach {name}   - Watch live (Ctrl+b d to exit)   ║
║    /loop kill {name}     - Stop the loop                   ║
║                                                            ║
║  Direct:                                                   ║
║    tmux capture-pane -t loop-{name} -p | tail -30          ║
║                                                            ║
╚════════════════════════════════════════════════════════════╝
```

## Subcommands

### /loop status
```bash
bash .claude/skills/tmux-loop/scripts/check-sessions.sh
```

### /loop attach NAME
Attach to session. Remind: `Ctrl+b` then `d` to detach.

### /loop kill NAME
Confirm completion status, then terminate.

## Key Principles

1. **Subagents for research** - Always spawn Explore agents to understand codebase before planning
2. **PRD is the source of truth** - Tasks come from a structured PRD, not ad-hoc
3. **prd.json is executable** - Each story has verifiable acceptance criteria
4. **prompt.md is standard** - Same instructions for every loop, tasks vary
5. **Fresh context per iteration** - Loop restarts Claude each iteration to avoid degradation
