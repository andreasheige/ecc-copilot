# Setup Guide

> Get ecc-copilot running in your project in under 5 minutes.

---

## Prerequisites

- **GitHub Copilot** — Pro+ or higher recommended (Pro+ unlocks all models including Opus, Sonnet, GPT-5.4)
- **VS Code** with GitHub Copilot Chat extension
- **Node.js** 18+ (for lefthook git hooks)
- **Git** repository initialized

### Plan Compatibility

| Feature | Free | Pro | Pro+ | Business | Enterprise |
|---------|------|-----|------|----------|------------|
| Agents & Skills | ✅ | ✅ | ✅ | ✅ | ✅ |
| Standard models (Sonnet, GPT-5.2) | — | — | ✅ | ✅ | ✅ |
| Premium models (Opus 4.6) | — | — | ✅ | ✅ | ✅ |
| Multi-model reviews (3+ models) | — | — | ✅ | ✅ | ✅ |
| Economy models (Haiku, GPT-5 mini) | ✅ | ✅ | ✅ | ✅ | ✅ |

The pipeline works on all plans but automatically falls back to available models. Multi-model parallel reviews require Pro+ or higher for full model diversity.

---

## Step 1 — Install the Package

### Option A: APM Install

```bash
cd your-project/
apm install YOUR_ORG/ecc-copilot
```

### Option B: Manual Copy

Copy the `.github/` folder into your project:

```bash
cp -r /path/to/ecc-copilot/.github/ your-project/.github/
```

This installs:
- `.github/agents/` — 35 agent definitions
- `.github/skills/` — 13 skill definitions with examples and templates
- `.github/instructions/` — 3 instruction files (core, model-selection, pipeline-artifacts)
- `.github/copilot-instructions.md` — global Copilot instructions
- `.github/pipeline-artifacts/` — learnings and session directories

---

## Step 2 — Install Git Hooks

```bash
npm install -D lefthook
npx lefthook install
```

Copy the hook config if not already present:

```bash
cp /path/to/ecc-copilot/lefthook.yml your-project/lefthook.yml
```

This activates:
- `pre-commit`: lint + format + secret scan + typecheck + console.log detection
- `pre-push`: full quality gate suite
- `commit-msg`: conventional commits validation

---

## Step 3 — Configure Your Project Instructions

Edit `.github/copilot-instructions.md` to add your project-specific details:

```markdown
## Project Stack
- Framework: [Next.js 15 / Express / Django / etc.]
- Language: [TypeScript / Python / Go / etc.]
- Database: [PostgreSQL / MongoDB / etc.]
- Styling: [Tailwind / CSS Modules / etc.]

## Project Conventions
- [Your coding standards here]
- [Your naming conventions here]
- [Your architecture patterns here]
```

Keep this file under 300 lines — it's loaded into every Copilot context.

---

## Step 4 — (Optional) Add Global Instructions

For settings that apply across all your projects:

```bash
cat >> ~/.copilot/copilot-instructions.md << 'EOF'
# Global Coding Standards

## Core Principles
- Always plan before implementing complex features
- Write tests before implementation (TDD, 80%+ coverage)
- Never hardcode secrets — use environment variables
- Prefer immutable patterns
- Functions < 50 lines, files < 800 lines
- Handle errors explicitly at every level

## Session Habits
- /resume at session start
- /review after code changes
- /compact when context gets heavy
- /share at session end
EOF
```

---

## Step 5 — (Optional) Configure MCP Servers

Add frequently used MCP servers to `~/.copilot/mcp-config.json`:

```json
{
  "mcpServers": {
    "context7": {
      "command": "npx",
      "args": ["-y", "@upstash/context7-mcp@latest"]
    },
    "sequential-thinking": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-sequential-thinking"]
    },
    "memory": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-memory"]
    },
    "playwright": {
      "command": "npx",
      "args": ["-y", "@playwright/mcp"]
    }
  }
}
```

**Hard limit: 10 active MCPs simultaneously** (~500 tokens per tool schema).

---

## Verify Installation

### Check agents are detected

In VS Code Copilot Chat, type `/agent` — you should see the full agent list including `architect`, `planner`, `code-reviewer`, etc.

### Check instructions are loaded

Ask Copilot: "What instructions do you have?" — it should reference ecc-core, model-selection, and pipeline-artifacts.

### Check hooks are active

```bash
npx lefthook run pre-commit
```

### Run a test pipeline

Ask Copilot:
```
@architect Review this codebase and tell me what you see.
```

The architect should scope the work, identify areas, and describe what agents it would dispatch.

---

## How It Works

### The Pipeline

Every non-trivial task flows through 7 stages:

```
User Request → architect (Stage 0)
    → analyser + work-item-creator (Stage 1)
    → parallel dev agents (Stage 2)
    → code-quality-reviewer + security-reviewer (Stage 3 — multi-model)
    → QA agents (Stage 4)
    → devsecops-deploy (Stage 5)
    → session-reporter (Stage 6)
```

### Model Selection

Agents automatically pick the right model for the job:

| Agent Type | Models | Cost |
|-----------|--------|------|
| Orchestration | Opus 4.6 | 3x |
| Code generation | Sonnet 4.5 + GPT-5.3-Codex | 1x |
| Testing/QA | Sonnet 4 + GPT-5.2 | 1x |
| Documentation | GPT-5.4 mini + GPT-5 mini | 0.33x / 0x |
| Build fixes | Haiku 4.5 + GPT-5 mini | 0.33x / 0x |
| Analysis | Gemini 3.1 Pro + Sonnet 4.5 | 1x |

### Multi-Model Reviews

Code reviews dispatch 3–5 parallel sub-reviewers with different models:

| PR Size | Reviewers |
|---------|-----------|
| ≤ 10 lines | 1 (single model) |
| 11–500 lines | 3 (Sonnet 4.5 + GPT-5.3-Codex + Gemini 3.1 Pro) |
| > 500 lines | 5 (+ Opus 4.6 + GPT-5.4) |

### Artifacts & Learning

Every agent logs invocations to `agent-log.jsonl`. Learnings persist across sessions in `.github/pipeline-artifacts/learnings/`. The `session-reporter` compiles cost/timing dashboards.

---

## Daily Workflow

```
Morning:   /resume → pick up where you left off
Feature:   @architect "implement feature X"  →  full pipeline runs
Quick fix: @build-error-resolver  →  fast fix with cheap model
Review:    @code-reviewer  →  multi-model parallel review
End:       /share → extract learnings → update instructions
```

### Session Habits

| When | Action |
|------|--------|
| Session start | `/resume` |
| Complex work | `/plan` or `Shift+Tab` → Plan mode |
| After code | `/review` |
| Context heavy | `/compact` |
| Session end | `/share` → extract learnings |

---

## Customization

### Adjust Model Tiers

Edit `.github/instructions/model-selection.instructions.md` to change model assignments per task type or add budget-conscious overrides.

### Add Project-Specific Agents

Create new `.github/agents/your-agent.agent.md` files:

```yaml
---
name: your-agent
description: What this agent does
tools: [read, edit, execute, search]
model: ["Claude Sonnet 4.5", "GPT-5.3-Codex", "Claude Sonnet 4"]
---

# Your Agent

Instructions for this agent...
```

### Add Project-Specific Skills

Create new `.github/skills/your-skill/SKILL.md` files following the existing pattern. Use sub-folders for examples, templates, and references.

### Disable Pipeline Stages

To skip stages (e.g., deploy), edit the `architect.agent.md` pipeline dispatch logic.

---

## Troubleshooting

| Problem | Solution |
|---------|----------|
| Agents not showing in `/agent` | Check files are in `.github/agents/` with `.agent.md` extension |
| Models falling back to wrong tier | Check plan supports the model — see model-selection matrix |
| Hooks not running | Run `npx lefthook install` again |
| Pipeline not dispatching | Invoke `@architect` explicitly — auto-dispatch depends on Copilot routing |
| Skills not loading | Say "load the `<skill-name>` skill" explicitly |
| Context too large | `/compact` at phase boundaries, keep instructions under 300 lines |

---

## File Structure Reference

```
.github/
├── agents/                          # 35 agent definitions
│   ├── architect.agent.md           # Pipeline orchestrator (Stage 0)
│   ├── analyser.agent.md            # Impact analysis (Stage 1)
│   ├── frontend.agent.md            # React/Next.js (Stage 2)
│   ├── backend.agent.md             # Server-side (Stage 2)
│   ├── code-quality-reviewer.agent.md  # Quality gate (Stage 3)
│   ├── security-reviewer.agent.md   # Security gate (Stage 3)
│   ├── qa-functional.agent.md       # QA gate (Stage 4)
│   ├── devsecops-deploy.agent.md    # Deploy (Stage 5)
│   ├── session-reporter.agent.md    # Reports (Stage 6)
│   └── ... (26 more)
├── instructions/
│   ├── ecc-core.instructions.md     # Core behavioral rules
│   ├── model-selection.instructions.md  # Model matrix + multi-model reviews
│   └── pipeline-artifacts.instructions.md  # Artifacts + logging
├── skills/
│   ├── tdd-workflow/                # TDD with examples/references sub-folders
│   ├── blueprint/                   # Feature planning
│   ├── api-design/                  # API patterns with examples
│   ├── verification-loop/           # Pre-PR quality gate
│   ├── scaffold-generator/          # Meta: generates scaffolding skills
│   ├── runbook-generator/           # Meta: generates debugging runbooks
│   └── ... (7 more)
├── pipeline-artifacts/
│   ├── learnings/                   # 8 topic files (persistent knowledge)
│   └── sessions/                    # Per-run artifacts + agent-log.jsonl
├── prompts/
│   └── generate-pipeline-agents.prompt.md
├── copilot-instructions.md          # Global project instructions
lefthook.yml                         # Git hooks config
```
