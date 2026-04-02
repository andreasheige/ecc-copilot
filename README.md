# ecc-copilot

> **Everything Claude Code (ECC) v1.9.0 — ported to GitHub Copilot.**
> Makes Copilot proactive, agent-first, and TDD-first — as close to Claude Code as possible.

---

## Install

```bash
apm install YOUR_ORG/ecc-copilot
```

Or add to your `apm.yml`:
```yaml
dependencies:
  apm:
    - YOUR_ORG/ecc-copilot#v1.9.0
```

Then install the git hooks:
```bash
npm install -D lefthook
npx lefthook install
```

---

## What This Does

Copilot out of the box is reactive — it answers what you ask. Claude Code with ECC is **proactive** — it spawns agents automatically, enforces TDD without being told, applies quality gates, and uses a library of skills.

This package replicates that behavior in Copilot by installing:

### Global Instructions (`ecc-core.instructions.md`)
The behavioral core — tells Copilot to:
- **Auto-spawn agents** based on task type (no prompting needed)
- **Enforce TDD** on every non-trivial change
- **Plan before executing** for multi-file changes
- **Apply security review** on auth/payment/PII code
- **Use skills** from the library automatically
- Use **Plan/Autopilot modes** correctly

### Agents (8)
Installed to `.github/agents/` — invoked automatically or via `/agent`.

| Agent | When Copilot invokes it |
|-------|------------------------|
| `planner` | Complex feature request (>1 file) |
| `architect` | Architectural decision needed |
| `code-reviewer` | Code just written or modified |
| `tdd-guide` | New feature or bug fix |
| `security-reviewer` | Auth, payments, PII, new API endpoints |
| `build-error-resolver` | Build or type error |
| `refactor-cleaner` | Dead code or large refactor |
| `doc-updater` | Documentation needs updating |

### Skills (11)
Installed to `.github/skills/` — reference by name or Copilot loads automatically.

| Skill | Purpose |
|-------|---------|
| `tdd-workflow` | Red → Green → Refactor with examples |
| `blueprint` | Feature planning before coding |
| `api-design` | REST/GraphQL API design patterns |
| `verification-loop` | Pre-PR quality gate (build → types → lint → tests) |
| `agentic-engineering` | AI-native engineering patterns |
| `autonomous-loops` | Loop patterns: sequential, continuous-PR, Ralphinho |
| `continuous-agent-loop` | v1.9 canonical loop skill |
| `ralphinho-rfc-pipeline` | RFC-driven DAG for large features |
| `plankton-code-quality` | Quality enforcement patterns |
| `architecture-decision-records` | ADR templates and workflow |
| `git-workflow` | Branch strategy, PR workflow, commit conventions |

### Hook Workarounds (`lefthook.yml`)
ECC has 25 hooks. Copilot has none. `lefthook.yml` replicates the critical ones as git hooks:

| ECC Hook | Git Hook Equivalent |
|---------|-------------------|
| `pre:bash:block-no-verify` | lefthook enforces hooks can't be skipped |
| `pre:bash:commit-quality` | `pre-commit`: lint + format + secret scan |
| `stop:format-typecheck` | `pre-commit`: typecheck staged files |
| `stop:check-console-log` | `pre-commit`: detect `console.log` |
| `post:quality-gate` | `pre-push`: full check suite |
| `commit-msg` convention | `commit-msg`: conventional commits validation |

---

## The Gap vs ECC

These ECC features have **no equivalent** in Copilot — handle them manually:

| ECC Feature | Manual Equivalent |
|------------|------------------|
| `session:start` — auto-load context | `/resume` at session start |
| `stop:session-end` — auto-persist | `/share` at session end → extract to instructions |
| `pre:observe:continuous-learning` | After session: `/share` → review → update instructions |
| Cost tracking | Copilot usage dashboard in GitHub |
| Desktop notifications | VS Code native notifications |

---

## Usage After Install

Copilot will now:

1. **Read** `ecc-core.instructions.md` on every session — behavioral rules always active
2. **Suggest** loading skills when relevant (or load them explicitly: "load the tdd-workflow skill")
3. **Auto-route** to agents based on task context
4. **Enforce** TDD, quality gates, and security review

**Manual habits to build** (replace ECC's auto hooks):
```
Session start  → /resume
Before complex work → /plan  (or Shift+Tab → Plan mode)
After writing code → /review
Context heavy → /compact
Session end → /share → extract learnings → update instructions
```

---

## Differences From ECC

| | Claude Code + ECC | Copilot + ecc-copilot |
|---|---|---|
| Skills | Auto-loaded by hooks | Reference by name or auto-suggested |
| Agents | Spawned by hooks automatically | Invoked via `/agent` or AI routing |
| Hooks | 25 automated hooks | 6 git hooks via lefthook |
| Session memory | Auto-persisted | Manual `/resume` / `/share` |
| Instinct learning | Automatic | Manual: `/share` → extract → instructions |
| Headless loop | `claude -p` bash loop | GitHub Actions + Copilot coding agent |
| Mode cycling | N/A | `Shift+Tab` (Interactive/Plan/Autopilot) |
| Parallel agents | `/loop` + devfleet | `/fleet` + git worktrees |
