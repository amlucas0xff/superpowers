---
name: brainstorming
description: "You MUST use this before any creative work - creating features, building components, adding functionality, or modifying behavior. Explores user intent, requirements and design before implementation."
model: opus
---

# Brainstorming Ideas Into Designs

## Overview

Help turn ideas into fully formed designs and specs through natural collaborative dialogue.

Start by understanding the current project context, then ask questions one at a time to refine the idea. Once you understand what you're building, present the design in small sections (200-300 words), checking after each section whether it looks right so far.

## Beads Detection

**At the start of brainstorming, detect if beads is active:**

```bash
# Check both conditions
if [ -d ".beads" ] && command -v bd &> /dev/null; then
  echo "BEADS_ACTIVE"
fi
```

- If **beads active**: The pipeline chains through `plan-to-epic` and `epic-executor`
- If **no beads**: The pipeline uses TodoWrite-based execution (standard flow)

**Pipeline with beads:**
```
Brainstorming -> Design Doc -> Write Plan -> plan-to-epic -> epic-executor
```

**Pipeline without beads:**
```
Brainstorming -> Design Doc -> Write Plan -> executing-plans (TodoWrite)
```

## The Process

**Understanding the idea:**
- Check out the current project state first (files, docs, recent commits)
- Ask questions one at a time to refine the idea
- Prefer multiple choice questions when possible, but open-ended is fine too
- Only one question per message - if a topic needs more exploration, break it into multiple questions
- Focus on understanding: purpose, constraints, success criteria

**Exploring approaches:**
- Propose 2-3 different approaches with trade-offs
- Present options conversationally with your recommendation and reasoning
- Lead with your recommended option and explain why

**Presenting the design:**
- Once you believe you understand what you're building, present the design
- Break it into sections of 200-300 words
- Ask after each section whether it looks right so far
- Cover: architecture, components, data flow, error handling, testing
- Be ready to go back and clarify if something doesn't make sense

## After the Design

**Documentation:**
- Write the validated design to `docs/plans/YYYY-MM-DD-<topic>-design.md`
- Use elements-of-style:writing-clearly-and-concisely skill if available
- Commit the design document to git

**Implementation (if continuing):**
- Ask: "Ready to set up for implementation?"
- Use superpowers:using-git-worktrees to create isolated workspace
- Use superpowers:writing-plans to create detailed implementation plan

### Beads-Active Pipeline Continuation

**After writing-plans completes (when beads is active):**

1. **Plan Review Checkpoint:**
   ```
   Implementation plan saved: docs/plans/YYYY-MM-DD-<topic>-plan.md

   Tasks: <N>
   Estimated complexity: <low/medium/high based on task count>

   Review the plan. Ready to create beads epic?
   ```
   Wait for user approval.

2. **Epic Creation:**
   - Use superpowers:plan-to-epic with the plan file
   - Optionally include design doc: `--design docs/plans/YYYY-MM-DD-<topic>-design.md`
   - Creates epic with tasks, dependencies, and acceptance criteria

3. **Execution:**
   - Ask: "Epic created: <epic-id>. Ready to start execution?"
   - Use superpowers:epic-executor to run tasks with subagent workflow
   - Two-stage review per task (spec compliance + code quality)
   - Loops until epic 100% complete

## Key Principles

- **One question at a time** - Don't overwhelm with multiple questions
- **Multiple choice preferred** - Easier to answer than open-ended when possible
- **YAGNI ruthlessly** - Remove unnecessary features from all designs
- **Explore alternatives** - Always propose 2-3 approaches before settling
- **Incremental validation** - Present design in sections, validate each
- **Be flexible** - Go back and clarify when something doesn't make sense

## Enhanced User Skills

If available, consider using these user skills during brainstorming:

- **sequential-thinking (user)** - Use the MCP tool for complex problems requiring systematic step-by-step reasoning with ability to revise thoughts and branch into alternative approaches
- **writing-for-llms (user)** - Apply research-backed prompt engineering principles when crafting design documents or specifications
- **gemini-cli-integration (user)** - Use for analyzing large codebases (50+ files) when understanding existing architecture before designing new features
- **frontend-design (user)** - When brainstorming involves UI/UX design, use this skill to create distinctive, production-grade frontend interfaces with high design quality

## Optional Follow-up Actions

After completing the design phase, ask the user if they would like to:

- **Create formal documentation**: "Would you like me to help co-author detailed documentation for this design?" If yes, spawn an agent using **doc-coauthoring (user)** skill to guide through a structured workflow for creating comprehensive documentation, proposals, or technical specs.
