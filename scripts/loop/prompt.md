# Autonomous Agent

## Workflow

1. Read `scripts/loop/prd.json` for stories
2. Read `scripts/loop/progress.txt` (check Codebase Patterns first)
3. Ensure you're on the correct branch (from `branchName`)
4. Pick highest priority story where `passes: false` 
5. Implement that ONE story

## Verification (ALL must pass before proceeding)

```bash
npm run typecheck
npm test
ubs
```

If ANY verification fails → fix it before marking done.

## After Verification Passes

6. Commit with detailed message:
   ```
   feat: [ID] - [Title]

   - What was added/changed
   - Key implementation details
   - Any notable decisions made
   ```
7. Update prd.json: set `passes: true` for this story
8. Append to progress.txt:

```
## [Date] - [Story ID]
- What was implemented
- Files changed
- Learnings/gotchas discovered
---
```

Add new patterns to **Codebase Patterns** section at top of progress.txt.

## Stop Condition

If ALL stories have `passes: true`:
```
<promise>COMPLETE</promise>
```

Otherwise, end normally after completing ONE story.
