---
name: epic-executor
description: Execute a beads epic using subagent-driven development - sequential task execution with two-stage review (spec compliance, then code quality). Requires beads (bd) to be installed.
---

# Epic Executor

Execute all tasks within a beads epic using subagent-driven development. Tasks run sequentially with two-stage review after each.

## Prerequisites

**Beads must be active:**
```bash
# Verify before proceeding
if [ -d ".beads" ] && command -v bd &> /dev/null; then
  echo "Beads active - proceeding"
else
  echo "ERROR: Beads not active. Cannot execute epics without beads."
  exit 1
fi
```

## Usage

```
/epic-executor <epic-id>
```

## Setup Phase

### 1. Validate Epic

```bash
bd show <epic-id> --json
bd list --parent=<epic-id> --json
```

If epic doesn't exist or is already 100% complete, inform user and stop.

### 2. Note Base SHA

Record git SHA before starting for code review context:

```bash
git rev-parse HEAD
```

## Execution Loop

Process one task at a time until epic complete:

### 1. Check Completion

```bash
bd list --parent=<epic-id> --status=open --json
```

If no open tasks remain, announce completion and stop.

### 2. Get Next Ready Task

```bash
bd ready --parent=<epic-id> --limit=1 --json
```

If no tasks ready but epic incomplete, check `bd blocked --parent=<epic-id> --json` and report what's blocking.

### 3. Claim Task

```bash
bd update <task-id> --status=in_progress --json
```

### 4. Dispatch Implementer Subagent

Use subagent-driven-development pattern. Dispatch a fresh subagent with:
- Full task description from bd show (the Description field)
- Task design context (the Design field)
- Epic context
- Instructions to ask questions before starting
- Self-review checklist before reporting back

**If subagent asks questions**: Answer them, then let them continue.

### 5. Spec Compliance Review

Dispatch spec reviewer subagent to verify implementation matches specification.

**Critical:** The reviewer must read actual code and compare to requirements line by line. It does NOT trust the implementer's self-report.

Report: APPROVED or NEEDS_CHANGES with specific file:line references.

### 6. Code Quality Review

**Only after spec compliance passes.**

Use `superpowers:code-reviewer` subagent. Focus on:
- Security issues
- Test coverage
- Best practices for the language/framework
- Performance concerns

### 7. Handle Review Results

**If either review finds issues:**
1. Dispatch fix subagent with specific issues to address
2. Re-run the review that found issues
3. Repeat until both reviews pass

**If both reviews pass:** Proceed to close.

### 8. Close Task

```bash
bd close <task-id> --reason="Implemented and verified" --json
```

### 9. Continue

Go back to step 1 for next task.

## Completion

When no open tasks remain:

```
Epic <epic-id> complete!

Summary:
- Tasks completed: N
- All implementations reviewed and verified
```

## Key Principles

1. **Sequential execution** - One task at a time, no parallel conflicts
2. **Fresh subagent per task** - No context pollution between tasks
3. **Two-stage review** - Spec compliance first, then code quality
4. **Fix before closing** - Tasks with review issues get fixed, not skipped
5. **Use beads, not TodoWrite** - Track with `bd update` and `bd close`
6. **Always use --json** - All `bd` commands use `--json` for reliable parsing

## Resumability

The key feature: when you come back and say `continue epic <id>`, the executor:
1. Checks `bd list --parent=<epic-id>` to see what's done
2. Gets next ready task
3. Continues from where it left off

No re-explanation needed. State survived in beads.

## Enhanced User Skills

If available, consider using these user skills during epic execution:

- **sequential-thinking (user)** - Use the MCP tool when executing complex steps that require systematic reasoning or when troubleshooting blocked tasks
- **code-reviewer (user)** - Use for the code quality review stage
