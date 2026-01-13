# Superpowers Fork

Personal fork of [obra/superpowers](https://github.com/obra/superpowers) - Claude Code skills plugin.

## Quick Reference

- **Upstream**: `https://github.com/obra/superpowers.git`
- **Fork**: `https://github.com/amlucas0xff/superpowers.git`
- **Marketplace Fork**: `https://github.com/amlucas0xff/superpowers-marketplace.git`
- **Purpose**: Customized superpowers with Opus model enforcement and user skill integrations

## Fork Customizations

1. **Model: Opus enforced** in all agents and commands
2. **User skills integrated** - References to `~/.claude/skills/` throughout
3. **Optional follow-up actions** - Agent spawning suggestions at workflow endpoints
4. **No `disable-model-invocation`** - AI can invoke `/brainstorm`, `/write-plan`, `/execute-plan` via Skill tool

---

## Installation Guide

### Fresh Install (New Machine)

```bash
# 1. Add the custom marketplace
/plugin marketplace add amlucas0xff/superpowers-marketplace

# 2. Install superpowers from your marketplace
/plugin install superpowers@amlucas-superpowers-marketplace

# 3. Verify installation
/plugin
# Should show: superpowers @ amlucas-superpowers-marketplace
```

### Migration from obra/superpowers-marketplace

If you previously had `superpowers@superpowers-marketplace` installed:

```bash
# Step 1: Remove the old marketplace (in Claude Code CLI)
/plugin marketplace remove superpowers-marketplace

# Step 2: Uninstall old superpowers
/plugin uninstall superpowers@superpowers-marketplace

# Step 3: Clean up stale cache and config (in terminal)
rm -rf ~/.claude/plugins/cache/superpowers-marketplace/superpowers

# Step 4: Remove stale references from config files
jq 'del(.enabledPlugins["superpowers@superpowers-marketplace"])' \
    ~/.claude/settings.json > /tmp/settings_clean.json && \
    mv /tmp/settings_clean.json ~/.claude/settings.json

jq 'del(.plugins["superpowers@superpowers-marketplace"])' \
    ~/.claude/plugins/installed_plugins.json > /tmp/installed_clean.json && \
    mv /tmp/installed_clean.json ~/.claude/plugins/installed_plugins.json

# Step 5: Add your marketplace and install (in Claude Code CLI)
/plugin marketplace add amlucas0xff/superpowers-marketplace
/plugin install superpowers@amlucas-superpowers-marketplace

# Step 6: Restart Claude Code
```

### Troubleshooting

**Error: "Plugin 'superpowers' not found in marketplace 'superpowers-marketplace'"**
- Cause: Stale reference in `settings.json` or `installed_plugins.json`
- Fix: Run the cleanup commands in Step 4 above

**Error: "Plugin already installed"**
- Cause: Same-name plugin conflict from different marketplace
- Fix: Uninstall the old one first, clean cache, then install yours

**Verification Commands**
```bash
# Check which marketplace superpowers is from
cat ~/.claude/plugins/installed_plugins.json | jq '.plugins | keys | map(select(contains("superpowers")))'
# Should show: ["superpowers@amlucas-superpowers-marketplace"]

# Check git remote of installed superpowers
cat ~/.claude/plugins/cache/amlucas-superpowers-marketplace/superpowers/*/.git/config | grep url
# Should show: url = https://github.com/amlucas0xff/superpowers.git

# Check for disable-model-invocation (should NOT be present)
grep -r "disable-model-invocation" ~/.claude/plugins/cache/amlucas-superpowers-marketplace/superpowers/*/commands/
# Should return nothing
```

### Automated Fix Script

Save this to `~/.claude/scripts/fix-superpowers-plugin.sh`:

```bash
#!/bin/bash
# Fix superpowers plugin - remove obra's version, keep amlucas's fork
set -e

echo "=== Fixing Superpowers Plugin Installation ==="

# Remove old superpowers cache
echo "[1/4] Removing old superpowers cache..."
rm -rf ~/.claude/plugins/cache/superpowers-marketplace/superpowers

# Remove from installed_plugins.json
echo "[2/4] Cleaning installed_plugins.json..."
if command -v jq &> /dev/null; then
    jq 'del(.plugins["superpowers@superpowers-marketplace"]) | del(.plugins["double-shot-latte@superpowers-marketplace"])' \
        ~/.claude/plugins/installed_plugins.json > /tmp/installed_clean.json
    mv /tmp/installed_clean.json ~/.claude/plugins/installed_plugins.json
fi

# Disable in settings.json
echo "[3/4] Cleaning settings.json..."
if command -v jq &> /dev/null; then
    jq 'del(.enabledPlugins["superpowers@superpowers-marketplace"]) | del(.enabledPlugins["double-shot-latte@superpowers-marketplace"])' \
        ~/.claude/settings.json > /tmp/settings_clean.json
    mv /tmp/settings_clean.json ~/.claude/settings.json
fi

# Verify
echo "[4/4] Verifying..."
if [ -d ~/.claude/plugins/cache/amlucas-superpowers-marketplace/superpowers ]; then
    echo "OK: amlucas-superpowers-marketplace installed"
    REMOTE=$(cat ~/.claude/plugins/cache/amlucas-superpowers-marketplace/superpowers/*/.git/config 2>/dev/null | grep url | head -1)
    echo "    $REMOTE"
else
    echo "WARNING: amlucas-superpowers-marketplace not found"
    echo "Run: /plugin install superpowers@amlucas-superpowers-marketplace"
fi

echo ""
echo "=== Complete - Restart Claude Code ==="
```

---

## Directory Structure

```
skills/           # Skill definitions (SKILL.md per skill)
agents/           # Agent definitions (code-reviewer.md)
commands/         # Slash commands (brainstorm, execute-plan, write-plan)
hooks/            # Session hooks
docs/             # Documentation and plans
tests/            # Test files
```

## Common Tasks

### Sync with upstream
```bash
git fetch upstream
git merge upstream/main
git push origin main
```

### Edit a skill
1. Edit `skills/<skill-name>/SKILL.md`
2. Commit with conventional format: `feat(skill-name): description`
3. Push to origin

### Update cache after changes
```bash
rm -rf ~/.claude/plugins/cache/superpowers
git clone https://github.com/amlucas0xff/superpowers.git ~/.claude/plugins/cache/superpowers
```

## Key Files to Know

| File | Purpose |
|------|---------|
| `skills/brainstorming/SKILL.md` | Design workflow before implementation |
| `skills/systematic-debugging/SKILL.md` | Four-phase debugging methodology |
| `skills/test-driven-development/SKILL.md` | TDD state machine |
| `skills/writing-skills/SKILL.md` | How to create new skills |
| `agents/code-reviewer.md` | Code review agent definition |

## Conventions

- **Commits**: Use conventional commit format (`feat:`, `fix:`, `docs:`)
- **Skills**: Follow TDD approach - test before writing
- **Model directives**: Use `model: opus` in frontmatter
