# Beads Bridge Hook

**SessionStart hook that bridges beads task tracking with superpowers skills.**

## What It Does

This hook runs at the start of every Claude Code session and:

1. **If beads is active** (`.beads/` directory exists + `bd` CLI available):
   - Runs `bd prime` to inject workflow context (~1-2k tokens)
   - Shows ready tasks, workflow instructions, project context

2. **If beads CLI is available but project not initialized**:
   - Suggests enabling beads for persistent task tracking
   - Explains benefits and how to enable

3. **If beads CLI is not installed**:
   - Silent exit (no message - user may not want beads)

## Installation

### Manual Installation

1. Open your Claude Code settings file:
   - **User level**: `~/.claude/settings.json`
   - **Project level**: `.claude/settings.json`

2. Add the hook to the `hooks.SessionStart` array:

```json
{
  "hooks": {
    "SessionStart": [
      {
        "type": "command",
        "command": "... (copy from hook.json)",
        "timeout": 10000
      }
    ]
  }
}
```

### Using hook-factory installer

```bash
cd ~/.claude/skills/hook-factory
python3 installer.py install /path/to/superpowers/hooks/beads-bridge user
```

## How It Works With Superpowers

```
Session Start
    |
    v
[beads-bridge hook]
    |
    +-- beads active? --> bd prime (inject context)
    |
    +-- bd available, no .beads? --> Suggest initialization
    |
    +-- no bd CLI? --> Silent exit
    |
    v
[superpowers skills detect beads]
    |
    +-- brainstorming --> writing-plans --> plan-to-epic --> epic-executor
```

## Configuration

The hook has a 10-second timeout which is sufficient for `bd prime` to complete.

To customize the suggestion message, edit the `echo` statements in hook.json.

## Troubleshooting

### Hook not running

1. Check Claude Code logs: `~/.claude/logs/`
2. Verify settings.json syntax is valid JSON
3. Restart Claude Code after installing

### bd prime fails silently

- The hook uses `2>/dev/null || exit 0` to fail silently
- This prevents session startup from being blocked
- Check `bd doctor` for beads health

### Want to disable suggestion message

Edit the hook to remove the `else` branch, or add a check for a specific file:

```bash
# Only suggest if a marker file exists
if [ -f ".suggest-beads" ]; then
    echo "..."
fi
```

## Related

- [Beads Documentation](https://github.com/steveyegge/beads)
- [superpowers:plan-to-epic](../skills/plan-to-epic/SKILL.md)
- [superpowers:epic-executor](../skills/epic-executor/SKILL.md)
