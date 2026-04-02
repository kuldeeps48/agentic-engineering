---
name: update-knowledge
description: >-
  Capture learnings from the current session and persist them across all knowledge stores: memory, skills, instructions, and command files.
  USE FOR: save learnings, update knowledge, persist insights, update instructions, update skills, update memory, capture session learnings, knowledge update, save what we learned, update docs from session.
  DO NOT USE FOR: creating new skills from scratch (use agent-customization skill), writing application code, code review (use code-review skill).
---

# Update Knowledge — Platform Backend

> **Purpose:** Take learnings from the current session and update all relevant knowledge stores so the insights persist across conversations and agents.

---

## 0. Before You Start — Gather Learnings

**Identify what was learned this session.** Look for:

1. **Bugs found during implementation or review** — patterns that were missed, edge cases discovered
2. **Rules that conflicted with behavior** — structural rules that were applied mechanically and caused issues
3. **New patterns established** — solutions that should be replicated in future features
4. **Corrections to existing guidance** — instructions that were wrong, incomplete, or misleading

Ask the user: "What are the key learnings from this session?" if not already clear.

---

## 1. Knowledge Stores — Where to Update

There are **6 knowledge stores**, each with a different purpose and format. Update only the ones where the learning is relevant.

| Store                       | Path                              | Format                          | Scope                       | When to Update                                |
| --------------------------- | --------------------------------- | ------------------------------- | --------------------------- | --------------------------------------------- |
| **User memory**             | `/memories/`                      | Bullet points, brief            | Cross-workspace, persistent | General workflow preferences, common mistakes |
| **Repo memory**             | `/memories/repo/`                 | JSON (create only)              | Repository-scoped           | Codebase conventions, verified practices      |
| **CLAUDE.md**               | `CLAUDE.md`                       | Markdown, uses "Slash Commands" | Claude Code sessions        | Always-on rules, gotchas, patterns            |
| **copilot-instructions.md** | `.github/copilot-instructions.md` | Markdown, uses "Skills"         | GitHub Copilot sessions     | Always-on rules, gotchas, patterns            |
| **Skills**                  | `.github/skills/<name>/SKILL.md`  | Markdown + YAML frontmatter     | Copilot, task-specific      | Deep-dive procedures, checklists              |
| **Commands**                | `.claude/commands/<name>.md`      | Plain markdown (NO frontmatter) | Claude Code, task-specific  | Deep-dive procedures, checklists              |

### Critical Format Differences

**`.github/skills/` (Copilot):**

- Lives in a folder: `.github/skills/<skill-name>/SKILL.md`
- **MUST** have YAML frontmatter with `name` and `description`
- Description should include `USE FOR:` and `DO NOT USE FOR:` keywords
- Referenced in `copilot-instructions.md` Skills Reference table as `[name](.github/skills/name/SKILL.md)`

**`.claude/commands/` (Claude Code):**

- Single file: `.claude/commands/<command-name>.md`
- **NO** YAML frontmatter — starts directly with content
- Referenced in `CLAUDE.md` Slash Commands Reference table as `/project:<command-name>`

**`CLAUDE.md` vs `copilot-instructions.md`:**

- Content is nearly identical but terminology differs:
  - CLAUDE.md says "Slash Commands" and `/project:<name>`
  - copilot-instructions.md says "Skills" and links to `SKILL.md` files
- **Both must be updated in sync** when changing always-on rules

---

## 2. Update Process

### Step 1: Classify the Learning

For each learning, determine:

- **Is it always-on?** → Update `CLAUDE.md` + `copilot-instructions.md` (Common Gotchas, Transaction Pattern, etc.)
- **Is it task-specific?** → Update the relevant skill + command (e.g., code-review, api-patterns)
- **Is it a personal workflow preference?** → Update `/memories/` only
- **Is it a codebase fact?** → Create `/memories/repo/<name>.json`

### Step 2: Update Each Store

For each relevant store:

1. **Read the current content** before editing — check what's already there to avoid duplicates
2. **Find the right section** — don't append randomly; place the learning in context
3. **Match the existing style** — bullet points, numbered lists, or prose as appropriate
4. **Keep it concise** — especially in memory files and always-on instructions

### Step 3: Cross-Reference Tables

If a new skill/command was created, update the reference tables:

- `copilot-instructions.md` → Skills Reference table
- `CLAUDE.md` → Slash Commands Reference table

### Step 4: Verify Consistency

- `CLAUDE.md` and `copilot-instructions.md` should have the same rules (different format references)
- Skills and their corresponding commands should have the same content (different frontmatter)
- Memory should not contradict instruction files

---

## 3. Templates

### New Gotcha Entry (for CLAUDE.md / copilot-instructions.md)

```
N. **Short title:** Explanation of the gotcha — what goes wrong and why. Include the correct approach
```

### New Repo Memory (for /memories/repo/)

```json
{
  "subject": "Brief title",
  "fact": "The learning in 1-2 sentences with actionable implications",
  "citations": ["file/path1.py", "file/path2.py"],
  "reason": "Why this matters for future tasks",
  "category": "architecture|convention|testing|security"
}
```

### New User Memory Entry (for /memories/)

```
- **Short title:** One-line actionable insight
```

---

## 4. Checklist

Before finishing, verify:

- [ ] Learnings identified and classified
- [ ] `/memories/` updated (if general workflow insight)
- [ ] `/memories/repo/` created (if codebase-specific fact)
- [ ] `CLAUDE.md` updated (if always-on rule)
- [ ] `.github/copilot-instructions.md` updated (if always-on rule — keep in sync with CLAUDE.md)
- [ ] Relevant `.github/skills/<name>/SKILL.md` updated (if task-specific)
- [ ] Relevant `.claude/commands/<name>.md` updated (if task-specific — same content, no frontmatter)
- [ ] Reference tables updated (if new skill/command created)
- [ ] No contradictions between stores
