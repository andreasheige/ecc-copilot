# ECC → GitHub Copilot: Complete Summary

> **Scope:** Everything Claude Code (ECC) v1.9.0 investigation + full port to GitHub Copilot  
> **Output:** `ecc-copilot` APM package — installs 8 agents, 11 skills, 1 behavioral instruction set, and git hooks

---

## Table of Contents

1. [What We Investigated](#1-what-we-investigated)
2. [ECC in Numbers](#2-ecc-in-numbers)
3. [Core ECC Concepts](#3-core-ecc-concepts)
4. [The Translation Map](#4-the-translation-map)
5. [What Copilot Already Has (Built-In)](#5-what-copilot-already-has-built-in)
6. [The Irreducible Gaps](#6-the-irreducible-gaps)
7. [The ecc-copilot APM Package](#7-the-ecc-copilot-apm-package)
8. [The 15 Power Patterns (Adapted for Copilot)](#8-the-15-power-patterns-adapted-for-copilot)
9. [Setup: From Zero to Full Power](#9-setup-from-zero-to-full-power)
10. [Anti-Patterns](#10-anti-patterns)
11. [Related Artifacts](#11-related-artifacts)

---

## 1. What We Investigated

Two repositories:

**`everything-claude-code/` (ECC v1.9.0)**  
An Anthropic Hackathon winner (50K+ stars). NOT a codebase to build on — a **plugin collection** that installs into AI coding agents. Its purpose: make any AI coding agent proactive, TDD-first, agent-first, and quality-gated via a library of skills, agents, hooks, and rules.

**`src/` (Claude Code CLI source)**  
The TypeScript source of Claude Code itself — 2,100+ files covering the interactive AI terminal: commands, tools, memory, hooks, plugins, agents, skills, voice, vim, remote sessions, and more. This is the harness ECC plugs into.

**The proposal:** Identify everything worth having in ECC and port it to GitHub Copilot, since Copilot is our actual tool.

---

## 2. ECC in Numbers

| Component   | Count | Purpose                                                                 |
| ----------- | ----- | ----------------------------------------------------------------------- |
| Skills      | 158   | Domain workflow definitions (TDD, blueprinting, API design, loops…)     |
| Agents      | 36    | Specialized delegation targets (planner, reviewer, security, build…)    |
| Commands    | 68    | Slash command shims (legacy wrappers around skills)                     |
| Rules       | 60+   | Always-on guidelines (12 languages: TS, Python, Go, Rust…)              |
| Hooks       | 26+   | Trigger-based automations (PreToolUse, PostToolUse, Stop, SessionStart) |
| MCP configs | 30+   | External service integrations (GitHub, Supabase, Exa, Playwright…)      |
| Tests       | 1,723 | All passing                                                             |

**Core philosophy — 5 tenets:**

1. **Agent-First** — Route work to specialists; don't do everything in one context
2. **Test-Driven** — Tests before implementation, 80%+ coverage always
3. **Security-First** — Validate inputs, protect secrets, safe defaults
4. **Immutability** — Explicit state transitions, never mutate shared state
5. **Plan Before Execute** — Complex tasks: plan → implement → review → verify

---

## 3. Core ECC Concepts

### Skills vs Commands

- **Skills** = canonical workflow definitions (reusable, composable, context-rich)
- **Commands** = legacy slash-command shims pointing to skills
- **Use skills** — commands are compatibility wrappers during ECC migration

### Hooks vs Rules

- **Hooks** = trigger-based automations running when events fire (file edit, shell exec, session start/stop)
- **Rules** = always-on guidelines baked into the agent's system prompt

### Agents vs Skills

- **Agents** = specialized AI instances you delegate TO (narrower tools, focused context)
- **Skills** = workflow knowledge/prompts that augment the main context

### Memory Layers (ECC's 5-Layer Stack)

| Layer | Mechanism               | Scope                                    |
| ----- | ----------------------- | ---------------------------------------- |
| 1     | `CLAUDE.md` / `.rules/` | Always-on project instructions           |
| 2     | `~/.claude/skills/`     | Reusable workflow definitions            |
| 3     | Instinct system         | Auto-learned patterns via `/learn`       |
| 4     | MCP Memory server       | Cross-session persistent key-value store |
| 5     | Session files           | Full state snapshots (`/save-session`)   |

---

## 4. The Translation Map

| ECC Component                    | Copilot Equivalent                                           | Notes                                                |
| -------------------------------- | ------------------------------------------------------------ | ---------------------------------------------------- |
| `CLAUDE.md`                      | `.github/copilot-instructions.md`                            | Copilot reads `CLAUDE.md` natively too ✅            |
| `AGENTS.md`                      | `.github/agents/*.agent.md`                                  | Copilot reads `AGENTS.md` natively too ✅            |
| Skills                           | `.github/skills/*/SKILL.md`                                  | Identical format ✅                                  |
| Rules                            | `.github/instructions/*.instructions.md` with `applyTo` glob | ✅ Direct                                            |
| Agents                           | `.github/agents/*.agent.md`                                  | ✅ Direct                                            |
| MCP servers                      | `~/.copilot/mcp-config.json`                                 | Same JSON format ✅                                  |
| `/plan`                          | `/plan`                                                      | Built-in ✅                                          |
| `/code-review`                   | `/review`                                                    | Built-in ✅                                          |
| `/save-session`                  | `/session` + `/share`                                        | Built-in ✅                                          |
| `/resume-session`                | `/resume`                                                    | Built-in ✅                                          |
| `/context-budget`                | `/context`                                                   | Built-in ✅                                          |
| `/compact`                       | `/compact`                                                   | Built-in ✅                                          |
| `/pr`                            | `/pr`                                                        | Built-in ✅                                          |
| `/research`                      | `/research`                                                  | Built-in ✅                                          |
| `/loop` + devfleet               | `/fleet` + `/tasks`                                          | Built-in ✅                                          |
| `/delegate`                      | `/delegate`                                                  | Built-in ✅                                          |
| Hooks (PreToolUse, PostToolUse…) | ❌ None                                                      | Workaround: `lefthook.yml` git hooks                 |
| Instinct auto-learning           | ❌ None                                                      | Workaround: manual `/share` → extract → instructions |
| `claude -p` headless CLI         | ❌ None                                                      | Workaround: GitHub Actions + Copilot coding agent    |
| Session auto-persist on Stop     | ❌ None                                                      | Workaround: manual `/share` at session end           |

**Bottom line: ~90% of ECC value translates directly. The 10% gap is hooks + headless CLI.**

---

## 5. What Copilot Already Has (Built-In)

Commands that exist natively in Copilot CLI — no plugin needed:

| Command     | Purpose                                             |
| ----------- | --------------------------------------------------- |
| `Shift+Tab` | Cycle modes: Interactive → Plan → Autopilot         |
| `/plan`     | Plan before coding (use in Plan mode or as command) |
| `/review`   | Code review after changes                           |
| `/diff`     | Diff current changes                                |
| `/pr`       | Create a PR                                         |
| `/research` | GitHub + web research before implementing           |
| `/resume`   | Load previous session context                       |
| `/share`    | Export session to markdown                          |
| `/compact`  | Compact context window                              |
| `/context`  | View token usage                                    |
| `/fleet`    | Parallel subagent execution                         |
| `/tasks`    | View background tasks                               |
| `/delegate` | Send to GitHub Copilot coding agent → creates PR    |
| `/lsp`      | Language server configuration                       |
| `/mcp`      | Manage MCP server connections                       |
| `/ide`      | Connect to VS Code workspace                        |
| `/agent`    | Browse and invoke specialized agents                |

**Copilot-exclusive patterns (not in ECC):**

- `Shift+Tab` mode cycling (Interactive / Plan / Autopilot)
- `/delegate` → GitHub creates a proper PR autonomously
- `/fleet` parallel subagent execution with `/tasks` monitoring

---

## 6. The Irreducible Gaps

These ECC features have **no Copilot equivalent** and require workarounds:

### Hooks (26+ in ECC → 0 in Copilot)

ECC hooks fire automatically on events (file edit, shell command, session start/stop). Copilot has no hook API.

**Workaround:** `lefthook.yml` (git hooks) — covers the most critical:

| ECC Hook                   | lefthook Equivalent                           |
| -------------------------- | --------------------------------------------- |
| `pre:bash:block-no-verify` | Enforce hooks can't be bypassed               |
| `pre:bash:commit-quality`  | `pre-commit`: lint + format + secret scan     |
| `stop:format-typecheck`    | `pre-commit`: typecheck staged files          |
| `stop:check-console-log`   | `pre-commit`: detect `console.log`            |
| `post:quality-gate`        | `pre-push`: full check suite                  |
| `commit-msg`               | `commit-msg`: conventional commits validation |

### Instinct Auto-Learning

ECC's instinct system learns patterns from sessions automatically via `/learn-eval` → `/evolve` → `/rules-distill`. No Copilot equivalent.

**Workaround:** Manual process — after every significant session:

```
/share  →  review exported markdown  →  extract useful patterns  →  add to copilot-instructions.md
```

### Headless Loop (`claude -p`)

ECC enables non-interactive scripted loops: `claude -p "do this" | next step`. No Copilot CLI equivalent.

**Workaround:** GitHub Actions + Copilot coding agent. Create an issue, assign to `@copilot`, it creates a PR. Chain multiple issues for pipeline-like behavior. See `ralph-copilot/` for the full implementation.

### Session Auto-Persist

ECC's Stop hook auto-saves session state on exit. No Copilot equivalent.

**Workaround:** End every session manually with `/share`.

---

## 7. The ecc-copilot APM Package

The deliverable: an APM package that makes Copilot behave like Claude Code out of the box.

**Location:** `/Users/andreasheige/Dev/ecc-copilot/`

### Install

```bash
apm install YOUR_ORG/ecc-copilot
npm install -D lefthook && npx lefthook install
```

### What Gets Installed

**Instructions (1 file → `.github/instructions/`)**

- `ecc-core.instructions.md` — the behavioral core: proactive agent dispatch table, TDD enforcement, security rules, planning requirements, mode usage, session habits, prohibited patterns, parallel execution

**Agents (8 files → `.github/agents/`)**

| Agent                  | Auto-invoked when                      |
| ---------------------- | -------------------------------------- |
| `planner`              | Complex feature request (>1 file)      |
| `architect`            | Architectural decision needed          |
| `code-reviewer`        | Code just written or modified          |
| `tdd-guide`            | New feature or bug fix                 |
| `security-reviewer`    | Auth, payments, PII, new API endpoints |
| `build-error-resolver` | Build or type error                    |
| `refactor-cleaner`     | Dead code or large refactor            |
| `doc-updater`          | Documentation needs updating           |

**Skills (11 directories → `.github/skills/`)**

| Skill                           | Purpose                                            |
| ------------------------------- | -------------------------------------------------- |
| `tdd-workflow`                  | Red → Green → Refactor with examples               |
| `blueprint`                     | Feature planning before coding                     |
| `api-design`                    | REST/GraphQL API design patterns                   |
| `verification-loop`             | Pre-PR quality gate (build → types → lint → tests) |
| `agentic-engineering`           | AI-native engineering patterns                     |
| `autonomous-loops`              | Loop patterns adapted for Copilot + GitHub Actions |
| `continuous-agent-loop`         | Canonical Copilot loop skill                       |
| `ralphinho-rfc-pipeline`        | RFC-driven DAG for large features                  |
| `plankton-code-quality`         | Quality enforcement via lefthook                   |
| `architecture-decision-records` | ADR templates and workflow                         |
| `git-workflow`                  | Branch strategy, PR workflow, commit conventions   |

**Git hooks (`lefthook.yml`)**
6 hooks covering the critical ECC automations — see [Section 6](#6-the-irreducible-gaps).

### Manual Habits to Build (Replace ECC's Auto Hooks)

```
Session start  →  /resume
Complex work   →  /plan  (or Shift+Tab → Plan mode)
After code     →  /review
Context heavy  →  /compact
Session end    →  /share → extract learnings → update instructions
```

---

## 8. The 15 Power Patterns (Adapted for Copilot)

### Pattern 1: Daily Development Loop

```
Morning:   /resume → /context
Feature:   /plan → TDD → implement → /review → /pr
End:       /share → extract patterns → update instructions
```

### Pattern 2: Model Routing

| Task                      | Model                             |
| ------------------------- | --------------------------------- |
| File search, simple edits | Haiku (fast, cheap)               |
| Multi-file implementation | Sonnet (best balance)             |
| Complex architecture      | Opus (deep reasoning)             |
| Security analysis         | Opus (can't miss vulnerabilities) |

### Pattern 3: Context Window Discipline

- Max 10 MCP servers active (~500 tokens per tool schema)
- Keep `copilot-instructions.md` under 300 lines
- Use `/compact` manually at phase boundaries — never mid-task
- `/context` to audit token usage

### Pattern 4: Subagent Architecture (Agent-First)

Never do complex work in one context. Route to agents:

| Trigger               | Agent                  |
| --------------------- | ---------------------- |
| Any code written      | `code-reviewer`        |
| Auth / payments / PII | `security-reviewer`    |
| Build fails           | `build-error-resolver` |
| Complex feature       | `planner`              |
| Architecture decision | `architect`            |

### Pattern 5: The Memory Stack (Copilot Edition)

```
Layer 1: .github/copilot-instructions.md   ← Always-on (< 300 lines)
Layer 2: .github/skills/                   ← Reusable workflow definitions
Layer 3: Manual /share → extract → update  ← Replaces ECC instinct system
Layer 4: MCP Memory server                 ← Cross-session key-value store
Layer 5: /share exports                    ← Session snapshots
```

### Pattern 6: Parallel Execution

| Need                     | Copilot approach                             |
| ------------------------ | -------------------------------------------- |
| 2-3 independent features | Git worktrees + separate Copilot sessions    |
| N tasks run by Copilot   | `/fleet` + `/tasks`                          |
| GitHub-assigned work     | `/delegate` → Copilot creates PR             |
| Overnight unattended     | GitHub Actions + `@copilot` issue assignment |

### Pattern 7: Hook Automation

Hooks don't exist in Copilot — use `lefthook.yml`. Install once:

```bash
npm install -D lefthook && npx lefthook install
```

### Pattern 8: PRP Workflow (Disciplined Features)

```
/plan <feature>         ← Plan with full codebase awareness
# Review and edit plan
implement               ← Execute the plan
/review                 ← Mandatory code review
/pr                     ← Create PR
```

### Pattern 9: Security First

Simon Willison's Lethal Trifecta: **Private data + Untrusted content + External comms = Exploitation**

Pre-commit checklist (enforced by lefthook):

- No hardcoded secrets
- All user inputs validated at boundaries
- SQL queries parameterized
- HTML output sanitized
- Auth checked on every protected route
- Rate limiting on public endpoints
- Error messages don't leak internals

### Pattern 10: Research Before Code

```
Need library / solution?
  → /research "topic"
  → /docs <library> (via Context7 MCP — no hallucination)
  → then implement
```

### Pattern 11: Session Handoff

```
End of session: /share → review file → extract patterns → update instructions
Next session:   /resume → /context → continue
```

### Pattern 12: Team Knowledge Sharing

Manual equivalent of ECC's instinct import/export:

- Senior dev curates `copilot-instructions.md` + skills library
- Share via APM package (`apm install org/team-copilot`)
- New team member gets all patterns instantly on `apm install`

### Pattern 13: Autonomous Loops (Copilot Edition)

ECC: `claude -p "do X"` → bash loop  
Copilot: GitHub Actions → issue → `@copilot` → PR → CI → merge → repeat

```bash
# Single autonomous task
gh issue create \
  --title "feat: add OAuth2 login" \
  --body "TDD. Read docs/auth-spec.md. Implement in src/auth/." \
  --assignee "@copilot" \
  --label "copilot"
```

For multi-step pipelines: see `ralph-copilot/` — full RFC/DAG orchestration.

### Pattern 14: MCP Configuration Strategy

```
Tier 1 (always on, no API key):
  github, sequential-thinking, memory, playwright, context7

Tier 2 (per-project):
  exa (search), supabase, vercel, firecrawl

Tier 3 (task-scoped, toggle as needed):
  fal-ai (media), browserbase (cloud browser)
```

**Hard limit: 10 active MCPs simultaneously.**

### Pattern 15: Keyboard Shortcuts

| Shortcut      | Effect                                      |
| ------------- | ------------------------------------------- |
| `Shift+Tab`   | Cycle modes: Interactive → Plan → Autopilot |
| `Shift+Enter` | Multi-line input                            |
| `Ctrl+U`      | Delete entire line                          |
| `Esc Esc`     | Interrupt / restore                         |
| `Tab`         | Toggle thinking display                     |
| `@filename`   | Reference a file                            |

---

## 9. Setup: From Zero to Full Power

### Step 1 — Install the ecc-copilot APM package

```bash
# In any project repo
apm install YOUR_ORG/ecc-copilot

# Activate git hooks
npm install -D lefthook
npx lefthook install
```

### Step 2 — Add global instructions

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
- /share at session end → extract learnings → update this file
EOF
```

### Step 3 — Configure Tier-1 MCPs

Add to `~/.copilot/mcp-config.json`:

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
    "playwright": { "command": "npx", "args": ["-y", "@playwright/mcp"] }
  }
}
```

### Step 4 — Add project-level instructions

Create `.github/copilot-instructions.md` in each project with:

- Tech stack
- Coding conventions
- Architecture decisions
- Forbidden patterns

### Step 5 — Build the manual learning habit

After every significant session:

1. `/share` — export session
2. Review the export
3. Extract 1-3 useful patterns
4. Add to `~/.copilot/copilot-instructions.md` (global) or project instructions
5. Commit scoped instructions to the repo

---

## 10. Anti-Patterns

| Anti-Pattern                             | Why It Hurts                              | Fix                                             |
| ---------------------------------------- | ----------------------------------------- | ----------------------------------------------- |
| Everything in one massive prompt         | Context explodes, quality drops           | Decompose; use agents                           |
| Skip code review "just this once"        | Bugs accumulate                           | Mandatory: `code-reviewer` after every change   |
| 20+ MCPs active all the time             | Destroys context window                   | Max 10, project-scoped                          |
| Never running `/share` + extracting      | Repeat same mistakes forever              | End of every session                            |
| Using the strongest model for everything | High cost, no quality gain                | Match model to task complexity                  |
| No session saves                         | Lose hours of context overnight           | `/share` always                                 |
| Manual `/compact` disabled               | Compacts mid-task, loses critical context | Compact manually at phase boundaries            |
| Same agent generates + reviews           | Author bias misses obvious issues         | Always separate generator and reviewer roles    |
| No file ownership in parallel work       | Agents conflict on same files             | Assign explicit scope per worktree              |
| Vague prompts                            | Vague output                              | Use `/plan` first; provide instructions context |
| Skipping `lefthook install`              | Quality gates not active                  | Run `npx lefthook install` once per repo        |

---

## 11. Related Artifacts

### `ecc-copilot/` — The Core APM Package

Makes Copilot behave like Claude Code. Install into any project.  
`/Users/andreasheige/Dev/ecc-copilot/`

### `cos-copilot-apm/` — COS Frontend APM Package

Project-specific APM package for the cos-frontend monorepo (Next.js 15, TypeScript, Tailwind, Storyblok, Centra). Ports the existing `.github/copilot-instructions.md` (27 rules), 11 scoped instructions, 15 skills, and 4 agents into a distributable APM package.  
`/Users/andreasheige/Dev/cos-copilot-apm/`

### `ralph-copilot/` — Autonomous Loop Engine

GitHub Actions implementation of ECC's RFC/DAG loop and continuous PR loop patterns. Also handles ServiceNow incident intake: SNOW webhook → Jira ticket → GitHub Issue → `@copilot` assignment → PR → merge.  
`/Users/andreasheige/Dev/ralph-copilot/`

### `docs/` — Full Investigation Reference

| File                               | Contents                                        |
| ---------------------------------- | ----------------------------------------------- |
| `00-overview.md`                   | ECC architecture, tenets, memory layers         |
| `01-skills-catalog.md`             | All 158 skills categorized + top 20 deep-dives  |
| `02-agents-playbook.md`            | All 36 agents + orchestration patterns          |
| `03-commands-reference.md`         | All 68 commands with examples                   |
| `04-memory-and-context.md`         | Memory systems, token budget, instinct learning |
| `05-parallel-execution.md`         | Parallel execution: dmux, loops, GAN pattern    |
| `06-mcp-and-tools.md`              | MCP server setup + integrations catalog         |
| `07-copilot-power-patterns.md`     | 15 power patterns master reference              |
| `08-ecc-to-copilot-translation.md` | Full ECC → Copilot translation guide            |
| `copilot-global-instructions.md`   | Global instructions template                    |
| `share-confluence.md`              | Confluence page draft for team                  |
| `share-teams-post.md`              | Teams post draft for team                       |

---

> **The single most valuable habit:** `/share` at the end of every session. Extract patterns. Add to instructions. Repeat. This is the manual equivalent of ECC's instinct flywheel — and it compounds over time.
