# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Superpowers** is a Claude Code plugin that provides a comprehensive skills library of proven techniques, patterns, and workflows. The plugin delivers automatic integration of skills covering testing (TDD, async patterns), debugging (systematic approaches, root cause tracing), collaboration (brainstorming, planning, code review), and meta skills (creating and testing skills).

**Key Architecture Principle:** This is a plugin that primarily delivers *skills content*, not executable code. The plugin itself is a lightweight delivery mechanism - the value is in the skills that get injected into Claude Code sessions.

## Repository Structure

```
superpowers/
├── .claude-plugin/
│   └── plugin.json              # Plugin metadata and configuration
├── hooks/
│   ├── hooks.json               # Hook configuration for SessionStart
│   └── session-start.sh         # Injects using-superpowers skill at startup
├── commands/
│   ├── brainstorm.md            # Redirect to brainstorming skill
│   ├── write-plan.md            # Redirect to writing-plans skill
│   └── execute-plan.md          # Redirect to executing-plans skill
├── skills/                      # Skills library (21 skills)
│   ├── using-superpowers/       # Introduction to skills system
│   ├── test-driven-development/ # TDD methodology
│   ├── systematic-debugging/    # 4-phase debugging framework
│   ├── brainstorming/           # Socratic design refinement
│   ├── writing-skills/          # How to create skills (TDD for docs)
│   └── [18 other skills...]
└── lib/
    └── initialize-skills.sh     # Skills repo initialization (unused in v3.0+)
```

## Core Architecture Concepts

### Skills System (v3.0+)

**Migration from v2.x to v3.0:** The plugin moved from maintaining a separate skills repository to using Claude Code's first-party skills system. Skills now live directly in this repository under `skills/`.

**SessionStart Hook Flow:**
1. Hook triggers on startup/resume/clear/compact
2. `hooks/session-start.sh` reads `skills/using-superpowers/SKILL.md`
3. Content injected into session context as "EXTREMELY_IMPORTANT"
4. This establishes mandatory workflows for finding and using skills

### Skill Structure

Each skill follows a standardized format:

**Directory structure:**
```
skill-name/
  SKILL.md              # Main reference (required)
  supporting-file.*     # Only if needed (reusable tools, heavy reference)
```

**SKILL.md frontmatter (YAML):**
```yaml
---
name: skill-name-with-hyphens
description: Use when [specific triggering conditions] - [what it does, third person]
---
```

**Content sections:**
- Overview (core principle in 1-2 sentences)
- When to Use (symptoms and use cases)
- Implementation (patterns, code examples)
- Common Mistakes / Rationalizations
- Integration with Other Skills

### Slash Commands

Commands are **thin wrappers** that activate corresponding skills:
- `/brainstorm` → loads `skills/brainstorming/SKILL.md`
- `/write-plan` → loads `skills/writing-plans/SKILL.md`
- `/execute-plan` → loads `skills/executing-plans/SKILL.md`

Commands do NOT contain implementation - they only redirect to skills.

## Development Commands

### Testing the Plugin Locally

Since this is a content delivery plugin (no build step), testing involves:

1. **Install locally:**
   ```bash
   # Add local marketplace
   /plugin marketplace add local /path/to/superpowers/.claude-plugin
   /plugin install superpowers@local
   ```

2. **Test SessionStart hook:**
   ```bash
   # Start new session or run
   /clear
   # Verify using-superpowers skill content appears in context
   ```

3. **Test slash commands:**
   ```bash
   /brainstorm
   /write-plan
   /execute-plan
   # Verify each loads corresponding skill
   ```

### Git Workflow

```bash
# Check status
git status

# Create feature branch
git checkout -b feature-name

# After changes
git add .
git commit -m "Description

🤖 Generated with [Claude Code](https://claude.com/claude-code)

Co-Authored-By: Claude <noreply@anthropic.com>"

# Push to origin
git push origin feature-name

# Create PR to main
gh pr create --title "Title" --body "Summary"
```

### Working with Skills

**Creating new skills (REQUIRED workflow):**

Skills are TDD for documentation. The `writing-skills` skill defines the mandatory process:

1. **RED Phase:** Create pressure scenarios with subagents WITHOUT the skill, document baseline behavior
2. **GREEN Phase:** Write minimal skill addressing those specific failures
3. **REFACTOR Phase:** Identify new rationalizations, close loopholes, re-test

**Never create untested skills.** The testing methodology is in `skills/testing-skills-with-subagents/SKILL.md`.

**Skill naming conventions:**
- Use kebab-case: `test-driven-development`, `using-git-worktrees`
- Verb-first or gerunds: `creating-skills` > `skill-creation`
- Active voice describing the action
- Name field must match directory name (lowercase)

## Critical Development Principles

### The Iron Laws

Three non-negotiable rules that appear across multiple skills:

1. **TDD:** "NO PRODUCTION CODE WITHOUT A FAILING TEST FIRST"
2. **Debugging:** "NO FIXES WITHOUT ROOT CAUSE INVESTIGATION FIRST"
3. **Skills:** "NO SKILL WITHOUT A FAILING TEST FIRST"

These rules are **rigid**, not flexible. The skills explicitly forbid rationalization and list specific excuses to counter.

### Mandatory Workflows

When a skill exists for your task, using it becomes **required**:

1. Before ANY task, check for relevant skills
2. If skill exists, announce you're using it: "I'm using [skill-name] to [purpose]"
3. Follow it exactly
4. If skill has checklist, create TodoWrite todos for EACH item

**Don't rationalize:**
- "I remember this skill" → Skills evolve, read current version
- "This doesn't count as a task" → It counts, find and read skills
- "Workflow is overkill" → Specific instructions mean workflows matter MOST

### Philosophy Embedded in Skills

- **Test-Driven Development** - Write tests first, always (test-driven-development skill)
- **Systematic over ad-hoc** - Process over guessing (systematic-debugging skill)
- **Evidence over claims** - Verify before declaring success (verification-before-completion skill)
- **Bulletproofing against rationalization** - Skills explicitly counter known excuses

## Cross-Skill Dependencies

Skills reference each other with explicit requirement markers:

- `**REQUIRED BACKGROUND:**` - Prerequisites you must understand first
- `**REQUIRED SUB-SKILL:**` - Skills that must be used in workflow
- `**Complementary skills:**` - Optional but helpful related skills

**Example from systematic-debugging:**
- REQUIRED: `root-cause-tracing` (when error is deep in call stack)
- REQUIRED: `test-driven-development` (when creating failing test case)
- Complementary: `defense-in-depth`, `condition-based-waiting`

## Common Pitfalls

### When Editing Skills

1. **Don't edit skills without testing** - Same TDD rule applies to documentation
2. **Don't use @ syntax for cross-references** - Forces immediate loading, burns context
3. **Don't repeat content across skills** - Use cross-references with requirement markers
4. **Don't make skills verbose** - Frequently-loaded skills should be <200 words

### When Using the Plugin

1. **Don't create .claude-plugin artifacts** - Plugin metadata is version controlled
2. **Don't edit hooks directly in production** - Test locally first
3. **Don't add executable code to command files** - Commands are thin redirects only

## Version History Notes

**v3.0.1 (current):** Integrated with Claude Code's first-party skills system
**v2.0.x:** Used separate superpowers-skills repository (deprecated)
**v1.x:** Monolithic plugin with embedded skills (deprecated)

The v3.0 migration removed:
- `lib/initialize-skills.sh` - No longer clones external repo
- Personal skills overlay system - Replaced with direct editing
- Skills auto-update mechanism - Skills version with plugin

## Plugin Marketplace

Published at: https://github.com/obra/superpowers-marketplace

Users install via:
```bash
/plugin marketplace add obra/superpowers-marketplace
/plugin install superpowers@superpowers-marketplace
```

## Philosophy on Skill Quality

From `writing-skills/SKILL.md`:

**"Writing skills IS Test-Driven Development applied to process documentation."**

- If you didn't watch an agent fail without the skill, you don't know if it teaches the right thing
- Untested skills = untested code (unacceptable)
- Baseline testing reveals rationalizations that must be explicitly countered
- Skills that enforce discipline need to resist rationalization under pressure

## Support and Issues

- Issues: https://github.com/obra/superpowers/issues
- Marketplace: https://github.com/obra/superpowers-marketplace
- Blog post: https://blog.fsck.com/2025/10/09/superpowers/
