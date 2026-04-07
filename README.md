# ecc-copilot

> **Everything Claude Code (ECC) v1.9.0 — ported to GitHub Copilot.**
> A full multi-stage agent pipeline with TDD enforcement, multi-model reviews, and autonomous quality gates.

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

Copilot out of the box is reactive — it answers what you ask. This package makes it **proactive** — it spawns agents automatically, enforces TDD, coordinates a 7-stage pipeline, runs multi-model parallel reviews, and tracks artifacts across sessions.

### Architecture

```
User Request
    ↓
Stage 0: Orchestration (architect)
    ↓
Stage 1: Analysis (analyser → work-item-creator)
    ↓
Stage 2: Development (parallel agents by domain)
    ↓
Stage 3: Review Gates (code-quality + security — multi-model parallel)
    ↓
Stage 4: QA Gates (functional + integration + performance + contract)
    ↓
Stage 5: Deploy (devsecops-deploy)
    ↓
Stage 6: Report (session-reporter)
```

---

## Components

### Instructions (3 files)

| File | Scope | Purpose |
|------|-------|---------|
| `ecc-core.instructions.md` | `**` | Core behavioral rules: agent-first, TDD-first, security, quality standards |
| `model-selection.instructions.md` | `**` | Multi-vendor model matrix, plan-based availability, multi-model review protocol |
| `pipeline-artifacts.instructions.md` | `.github/agents/**` | Artifact collection, invocation logging (JSONL), memory progression tiers |

### Agents (34)

All agents live in `.github/agents/`. The `architect` is the entry point — it dispatches everything else.

#### Stage 0 — Orchestration
| Agent | Model Tier | Purpose |
|-------|-----------|---------|
| `architect` | Opus 4.6 (3x) | Pipeline orchestrator, entry point for all non-trivial tasks |
| `planner` | Opus 4.6 (3x) | Detailed planning for complex features |

#### Stage 1 — Analysis
| Agent | Model Tier | Purpose |
|-------|-----------|---------|
| `analyser` | Gemini 3.1 Pro (1x) | Impact analysis, affected file/component scanning |
| `work-item-creator` | GPT-5.4 mini (0.33x) | Epic → Story → Task breakdown with acceptance criteria |

#### Stage 2 — Development (parallel)
| Agent | Model Tier | Purpose |
|-------|-----------|---------|
| `frontend` | Sonnet 4.5 + GPT-5.3-Codex (1x) | React, Next.js, Tailwind, a11y |
| `backend` | Sonnet 4.5 + GPT-5.3-Codex (1x) | Routes, controllers, services, middleware |
| `api-expert` | Sonnet 4.5 + GPT-5.3-Codex (1x) | API contracts, OpenAPI, versioning |
| `database` | Sonnet 4.5 + GPT-5.3-Codex (1x) | Schema, migrations, query optimization |
| `ai-ml` | Sonnet 4.5 + GPT-5.3-Codex (1x) | LLM integration, embeddings, prompts |
| `payments` | Sonnet 4.5 + GPT-5.3-Codex (1x) | Stripe, subscriptions, PCI compliance |
| `rag-embedding` | Sonnet 4.5 + GPT-5.3-Codex (1x) | Vector stores, chunking, retrieval |
| `devsecops` | Sonnet 4.5 + GPT-5.3-Codex (1x) | CI/CD, Docker, Terraform, secret scanning |
| `search-discovery` | Sonnet 4.5 + GPT-5.3-Codex (1x) | Full-text search, facets, ranking |
| `notification-comms` | Sonnet 4.5 + GPT-5.3-Codex (1x) | Email, push, in-app, SMS |
| `analytics-data-layer` | Sonnet 4.5 + GPT-5.3-Codex (1x) | Event tracking, data layer contracts |
| `compliance-gdpr` | Sonnet 4.5 + GPT-5.3-Codex (1x) | Consent, data retention, right-to-erasure |
| `localization` | Sonnet 4.5 + GPT-5.3-Codex (1x) | i18n, translation keys, RTL support |
| `dependency-supply-chain` | Sonnet 4 + GPT-5.2 (1x) | Package audits, license compliance, SBOM |

#### Stage 3 — Review Gates (multi-model parallel)
| Agent | Model Tier | Purpose |
|-------|-----------|---------|
| `code-reviewer` | Multi-model dispatch | General code review, 3–5 parallel sub-reviewers |
| `code-quality-reviewer` | Multi-model dispatch | Quality gate (PASS/FAIL), blocks pipeline |
| `security-reviewer` | Multi-model dispatch | Security gate (PASS/FAIL), strictest aggregation |

#### Stage 4 — QA Gates
| Agent | Model Tier | Purpose |
|-------|-----------|---------|
| `qa-functional` | Sonnet 4 + GPT-5.2 (1x) | Business logic verification |
| `qa-integration-e2e` | Sonnet 4 + GPT-5.2 (1x) | Integration + Playwright E2E tests |
| `qa-performance` | Sonnet 4 + GPT-5.2 (1x) | Performance budgets, Lighthouse, bundle size |
| `qa-automation-runner` | Sonnet 4 + GPT-5.2 (1x) | Test suite orchestrator, coverage aggregation |
| `contract-testing` | Sonnet 4 + GPT-5.2 (1x) | Consumer-driven contract tests |

#### Stage 5 — Deploy
| Agent | Model Tier | Purpose |
|-------|-----------|---------|
| `devsecops-deploy` | Sonnet 4.5 + GPT-5.3-Codex (1x) | Deployment execution, smoke tests, rollback |

#### Stage 6 — Report
| Agent | Model Tier | Purpose |
|-------|-----------|---------|
| `session-reporter` | GPT-5.4 mini (0.33x) | Cost/timing dashboard from agent-log.jsonl |

#### Utility Agents (auto-invoked)
| Agent | Model Tier | Purpose |
|-------|-----------|---------|
| `tdd-guide` | Sonnet 4 + GPT-5.2 (1x) | TDD workflow enforcement |
| `build-error-resolver` | Haiku 4.5 + GPT-5 mini (0.33x/0x) | Fast build/type error fixes |
| `refactor-cleaner` | Sonnet 4 + GPT-5.2 (1x) | Dead code removal, consolidation |
| `doc-updater` | GPT-5.4 mini + GPT-5 mini (0.33x/0x) | Codemap and docs updates |
| `documentation` | GPT-5.4 mini + GPT-5 mini (0.33x/0x) | API docs, READMEs, ADRs, runbooks |
| `test-automation` | Sonnet 4 + GPT-5.2 (1x) | Test framework setup, fixtures, CI config |

### Multi-Model Review Protocol

Code reviews use parallel agents with different models for diverse perspective:

| PR Size | Reviewers | Models |
|---------|-----------|--------|
| ≤ 10 lines | 1 (single) | Primary model only |
| 11–500 lines | 3 (parallel) | Sonnet 4.5 + GPT-5.3-Codex + Gemini 3.1 Pro |
| > 500 lines | 5 (parallel) | + Opus 4.6 + GPT-5.4 |

Findings are tagged: `[UNANIMOUS]`, `[MAJORITY]`, `[SINGLE:<model>]`. Security gate uses strictest aggregation — any single CRITICAL from any model = FAIL.

### Skills (13)

Skills live in `.github/skills/`. Some include sub-folders with examples, templates, and references.

| Skill | Purpose |
|-------|---------|
| `tdd-workflow` | Red → Green → Refactor with examples and mock patterns |
| `blueprint` | Multi-session feature planning with adversarial review |
| `api-design` | REST/GraphQL patterns with multi-language examples |
| `verification-loop` | Pre-PR quality gate (build → types → lint → tests → coverage) |
| `agentic-engineering` | Model routing, eval-gated steps, AI-native patterns |
| `autonomous-loops` | Sequential, continuous-PR, and multi-agent DAG patterns |
| `continuous-agent-loop` | Failure-mode handling for agent loops |
| `ralphinho-rfc-pipeline` | RFC-driven DAG for large features (>3 PRs) |
| `plankton-code-quality` | Auto-formatting, linting, AI-powered fixes |
| `architecture-decision-records` | ADR templates and lifecycle |
| `git-workflow` | Branch strategy, PR templates, commit conventions |
| `scaffold-generator` | Meta-skill: scans codebase → generates scaffolding skills |
| `runbook-generator` | Meta-skill: scans error handling → generates debugging runbooks |

### Model Selection Matrix

Agents automatically select models based on task type and cost tier:

| Cost Tier | Multiplier | Use Case | Example Models |
|-----------|-----------|----------|----------------|
| Free (0x) | 0 | Build fixes, docs | GPT-4.1, GPT-5 mini |
| Budget (0.25x) | 0.25 | Fast triage | Grok Code Fast 1 |
| Economy (0.33x) | 0.33 | Build fixes, docs, reports | Haiku 4.5, Gemini 3 Flash, GPT-5.4 mini |
| Standard (1x) | 1 | Code gen, testing, reviews | Sonnet 4.5, GPT-5.3-Codex, Gemini 3.1 Pro |
| Premium (3x) | 3 | Orchestration, security | Opus 4.6 |

Supports all GitHub Copilot models across Anthropic, OpenAI, Google, and xAI. Plan-based fallback: tries recommended model → drops to next tier if unavailable.

### Pipeline Artifacts & Learning

```
.github/pipeline-artifacts/
├── learnings/           # Persistent cross-session knowledge
│   ├── architecture.md
│   ├── backend.md
│   ├── code-quality.md
│   ├── frontend.md
│   ├── general.md
│   ├── performance.md
│   ├── security.md
│   └── testing.md
└── sessions/            # Per-run artifacts + agent-log.jsonl
```

**Invocation logging**: Every agent appends JSONL events (`start`/`end`) to `agent-log.jsonl` with model, stage, timing, tool calls, and findings. The `session-reporter` reads this to compile dashboards.

**Memory progression**: Markdown (now) → JSONL (>200 lines) → Vector DB (>500 entries) — auto-detected.

### Hook Workarounds (`lefthook.yml`)

| ECC Hook | Git Hook Equivalent |
|---------|-------------------|
| `pre:bash:block-no-verify` | lefthook enforces hooks can't be skipped |
| `pre:bash:commit-quality` | `pre-commit`: lint + format + secret scan |
| `stop:format-typecheck` | `pre-commit`: typecheck staged files |
| `stop:check-console-log` | `pre-commit`: detect `console.log` |
| `post:quality-gate` | `pre-push`: full check suite |
| `commit-msg` convention | `commit-msg`: conventional commits validation |

---

## Usage

Copilot reads the instructions automatically. For any non-trivial task, the `architect` agent takes over and runs the full pipeline.

**Session habits:**
```
Session start    → /resume
Complex work     → /plan  (or Shift+Tab → Plan mode)
After code       → /review
Context heavy    → /compact
Session end      → /share → extract learnings → update instructions
```

**Agent invocation:**
```
/agent architect    — Full pipeline for a feature
/agent planner      — Detailed planning only
/agent code-reviewer — Review current changes
/agent tdd-guide    — TDD workflow for current task
```

**Skill loading:**
```
"load the blueprint skill"
"load the tdd-workflow skill"
"load the verification-loop skill"
```

---

## The Gap vs ECC

| ECC Feature | Manual Equivalent |
|------------|------------------|
| `session:start` — auto-load context | `/resume` at session start |
| `stop:session-end` — auto-persist | `/share` at session end → extract to instructions |
| `pre:observe:continuous-learning` | After session: `/share` → review → update instructions |
| Cost tracking | Copilot usage dashboard + session-reporter agent |
| Desktop notifications | VS Code native notifications |
