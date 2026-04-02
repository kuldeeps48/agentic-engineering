# Agentic Engineering Pipeline

An interactive visualization of a multi-agent AI engineering pipeline that automates the full software development lifecycle — from architecture design through implementation to code review — using specialized, collaborating AI agents.

## 🔗 Live Demo

**[View the Pipeline Visualization](https://kuldeeps48.github.io/agentic-enginerring/)**

## Overview

The pipeline orchestrates **5 specialized agents** across **up to 11 context windows**, with every output independently verified by a second AI model before any code ships.

### Pipeline Stages

| Stage | Agent | Role |
|-------|-------|------|
| **1. Architecture** | Architect | Analyzes requirements, researches the codebase via Explore subagents, and produces a detailed design document — scrutinized by a GPT cross-model verifier |
| **2. Implementation** | Implementer | Follows the architecture document task-by-task, writing code, tests, and migrations. Stays alive throughout the entire pipeline |
| **3. Code Review** | Code Reviewer | Multi-pass review across 10 categories, cross-verified by GPT Scrutinizer in two independent rounds. Findings are auto-fixed by the Implementer, then re-reviewed from scratch |

### Agents

- **Architect** — Designs before code is written. Uses Explore subagents for codebase research and GPT Scrutinizer for design gap analysis
- **Implementer** — The only long-lived context. Executes tasks, invokes Code Reviewer as subagent, auto-fixes all findings, and escalates only if issues persist after 2 review cycles
- **Code Reviewer** — Runs a 6-step review workflow with two GPT Scrutinizer rounds for cross-model verification
- **Explore** — Lightweight read-only research subagent. Spawned for codebase exploration without consuming the parent's context budget
- **GPT Scrutinizer** — Independent cross-model verifier (GPT-based). Checks architecture documents and code review findings for accuracy and completeness

### Key Design Principles

- **Cross-model verification** — Every significant output (architecture docs, code reviews) is independently verified by a different AI model
- **Context isolation** — Subagents run in their own context windows and don't consume the parent's token budget
- **Auto-fix loop** — Code review findings are automatically fixed, then the entire review runs again from scratch on the fixed code
- **Two-cycle maximum** — If findings persist after 2 full review cycles, they're escalated to the user with an assessment rather than looping indefinitely

## Features of the Visualization

- Expandable agent cards with detailed step-by-step workflows
- Nested subagent call visualization (Explore, GPT Scrutinizer)
- Context lifecycle tracking (which contexts stay alive vs. die after use)
- Token budget breakdown showing how the Implementer stays within ~200k limits
- Executive summary view with a 3-stage flow diagram
- Fully responsive, self-contained HTML — no build step or dependencies

## Usage

Open `index.html` in any browser, or visit the [GitHub Pages deployment](https://kuldeeps48.github.io/agentic-enginerring/).

## License

MIT
