---
name: autonomous-loops
description: "Patterns and architectures for autonomous GitHub Copilot loops — from simple sequential pipelines to RFC-driven multi-agent DAG systems."
origin: ECC
---

# Autonomous Loops Skill

> Compatibility note (v1.8.0): `autonomous-loops` is retained for one release.
> The canonical skill name is now `continuous-agent-loop`. New loop guidance
> should be authored there, while this skill remains available to avoid
> breaking existing workflows.

Patterns, architectures, and reference implementations for running GitHub Copilot autonomously in loops. Covers everything from simple GitHub Actions pipelines to full RFC-driven multi-agent DAG orchestration.

## When to Use

- Setting up autonomous development workflows that run without human intervention
- Choosing the right loop architecture for your problem (simple vs complex)
- Building CI/CD-style continuous development pipelines
- Running parallel agents with merge coordination
- Implementing context persistence across loop iterations
- Adding quality gates and cleanup passes to autonomous workflows

## Loop Pattern Spectrum

From simplest to most sophisticated:

| Pattern | Complexity | Best For |
|---------|-----------|----------|
| [Sequential Pipeline](#1-sequential-pipeline-github-actions) | Low | Daily dev steps, scripted workflows |
| [NanoClaw REPL](#2-nanoclaw-repl) | Low | ⚠️ Not available in Copilot |
| [Infinite Agentic Loop](#3-infinite-agentic-loop) | Medium | Parallel content generation, spec-driven work |
| [Ralph-Copilot Loop](#4-ralph-copilot-loop) | Medium | Multi-day iterative projects with CI gates |
| [De-Sloppify Pattern](#5-the-de-sloppify-pattern) | Add-on | Quality cleanup after any Implementer step |
| [Ralphinho / RFC-Driven DAG](#6-ralphinho--rfc-driven-dag-orchestration) | High | Large features, multi-unit parallel work with merge queue |

---

## 1. Sequential Pipeline (GitHub Actions)

**The simplest loop.** Break daily development into a sequence of GitHub Actions jobs. Each job is a focused step that assigns a GitHub Issue to `@copilot` — the Copilot coding agent picks it up and creates a PR automatically.

### Core Insight

> If you can't figure out a loop like this, it means you can't even drive the agent to fix your code in interactive mode.

In Copilot, `claude -p "..."` does not exist. The equivalent is to create a GitHub Issue with a detailed task description and assign it to `@copilot`. The Copilot coding agent picks it up and creates a PR automatically. Chain multiple issues (or wait for each PR to merge) to build a pipeline.

```bash
#!/bin/bash
# daily-dev.sh — Sequential pipeline for a feature branch (Copilot via GitHub Actions)

set -e

# Step 1: Implement the feature
gh issue create \
  --title "feat: implement OAuth2 login" \
  --body "Read docs/auth-spec.md. Implement OAuth2 login in src/auth/. Write tests first (TDD). Do NOT create any new documentation files." \
  --assignee "@copilot" \
  --label "copilot"
# Copilot coding agent picks it up and creates a PR automatically

# Step 2: De-sloppify (cleanup pass) — create after step 1 PR is merged
gh issue create \
  --title "chore: de-sloppify OAuth2 implementation" \
  --body "Review all files changed in the OAuth2 PR. Remove any unnecessary type tests, overly defensive checks, or testing of language features. Keep real business logic tests. Run the test suite after cleanup." \
  --assignee "@copilot" \
  --label "copilot"

# Step 3: Verify — create after step 2 PR is merged
gh issue create \
  --title "chore: verify OAuth2 build and tests" \
  --body "Run the full build, lint, type check, and test suite. Fix any failures. Do not add new features." \
  --assignee "@copilot" \
  --label "copilot"
```

> ⚠️ **Note:** `claude -p` (non-interactive CLI) does not exist in GitHub Copilot. The above `gh issue create --assignee @copilot` pattern is the closest equivalent. Each issue becomes an autonomous coding agent session that produces a PR.

### Key Design Principles

1. **Each step is isolated** — A separate Copilot agent session per issue means no context bleed between steps.
2. **Order matters** — Issues should be created and assigned sequentially (or wait for merge before next). Each builds on the filesystem state left by the previous PR.
3. **Negative instructions are dangerous** — Don't say "don't test type systems." Instead, add a separate cleanup step (see [De-Sloppify Pattern](#5-the-de-sloppify-pattern)).
4. **Exit codes propagate** — In a GitHub Actions workflow, `set -e` stops the pipeline on failure.

### Variations

**With task context via files:**
```bash
# Pass context via files referenced in the issue body
echo "Focus areas: auth module, API rate limiting" > SHARED_TASK_NOTES.md
git add SHARED_TASK_NOTES.md && git commit -m "chore: add task notes"

gh issue create \
  --title "chore: work through SHARED_TASK_NOTES.md priorities" \
  --body "Read SHARED_TASK_NOTES.md for priorities. Work through them in order. Update the file when done." \
  --assignee "@copilot" \
  --label "copilot"
```

**With read-only analysis:**
```bash
# Analysis-only issue (instruct Copilot not to write code)
gh issue create \
  --title "audit: security review of codebase" \
  --body "Audit this codebase for security vulnerabilities. Do NOT modify any files. Write your findings to security-audit.md only." \
  --assignee "@copilot" \
  --label "copilot"
```

---

## 2. NanoClaw REPL

> ⚠️ **Not available in GitHub Copilot.** NanoClaw is a Claude Code-specific persistent REPL that calls `claude -p` synchronously with full conversation history stored in `~/.claude/claw/{session}.md`. There is no direct Copilot equivalent.
>
> **Alternatives:**
> - For interactive exploration: use Copilot Chat in your editor (maintains session context within the editor window)
> - For scripted automation: use the [Sequential Pipeline (GitHub Actions)](#1-sequential-pipeline-github-actions) pattern
> - For context persistence across sessions: use `SHARED_TASK_NOTES.md` as a shared context file committed to the repo

---

## 3. Infinite Agentic Loop

**A two-prompt system** that orchestrates parallel sub-agents for specification-driven generation. Developed by disler (credit: @disler).

### Architecture: Two-Prompt System

```
PROMPT 1 (Orchestrator)              PROMPT 2 (Sub-Agents)
┌─────────────────────┐             ┌──────────────────────┐
│ Parse spec file      │             │ Receive full context  │
│ Scan output dir      │  deploys   │ Read assigned number  │
│ Plan iteration       │────────────│ Follow spec exactly   │
│ Assign creative dirs │  N agents  │ Generate unique output │
│ Manage waves         │             │ Save to output dir    │
└─────────────────────┘             └──────────────────────┘
```

### The Pattern

1. **Spec Analysis** — Orchestrator reads a specification file (Markdown) defining what to generate
2. **Directory Recon** — Scans existing output to find the highest iteration number
3. **Parallel Deployment** — Launches N sub-agents, each with:
   - The full spec
   - A unique creative direction
   - A specific iteration number (no conflicts)
   - A snapshot of existing iterations (for uniqueness)
4. **Wave Management** — For infinite mode, deploys waves of 3-5 agents until context is exhausted

### Implementation via Copilot Skills / GitHub Actions

Create a `.github/agents/infinite.md` skill file or a GitHub Actions workflow:

```markdown
Parse the following arguments from $ARGUMENTS:
1. spec_file — path to the specification markdown
2. output_dir — where iterations are saved
3. count — integer 1-N or "infinite"

PHASE 1: Read and deeply understand the specification.
PHASE 2: List output_dir, find highest iteration number. Start at N+1.
PHASE 3: Plan creative directions — each agent gets a DIFFERENT theme/approach.
PHASE 4: Deploy sub-agents in parallel (Task tool). Each receives:
  - Full spec text
  - Current directory snapshot
  - Their assigned iteration number
  - Their unique creative direction
PHASE 5 (infinite mode): Loop in waves of 3-5 until context is low.
```

**Invoke by loading the skill and asking Copilot:**
```
Load the infinite-agentic-loop skill. Use spec: specs/component-spec.md, output: src/, count: 5
Load the infinite-agentic-loop skill. Use spec: specs/component-spec.md, output: src/, count: infinite
```

### Batching Strategy

| Count | Strategy |
|-------|----------|
| 1-5 | All agents simultaneously |
| 6-20 | Batches of 5 |
| infinite | Waves of 3-5, progressive sophistication |

### Key Insight: Uniqueness via Assignment

Don't rely on agents to self-differentiate. The orchestrator **assigns** each agent a specific creative direction and iteration number. This prevents duplicate concepts across parallel agents.

---

## 4. Ralph-Copilot Loop

**A production-grade GitHub Actions workflow** that runs the Copilot coding agent in a continuous loop, creating PRs, waiting for CI, and merging automatically. Adapted from the Continuous Claude pattern by AnandChowdhary (credit: @AnandChowdhary) for GitHub Copilot via ralph-copilot.

### Core Loop

```
┌─────────────────────────────────────────────────────┐
│  RALPH-COPILOT ITERATION                            │
│                                                     │
│  1. Create branch (ralph-copilot/iteration-N)       │
│  2. Create GitHub Issue, assign to @copilot         │
│  3. (Optional) Reviewer pass — separate issue       │
│  4. Copilot commits changes + creates PR            │
│  5. Push + create PR (Copilot agent does this)      │
│  6. Wait for CI checks (poll gh pr checks)          │
│  7. CI failure? → Create fix issue, assign @copilot │
│  8. Merge PR (squash/merge/rebase)                  │
│  9. Return to main → repeat                         │
│                                                     │
│  Limit by: --max-runs N | --max-duration 2h         │
│            | completion signal in SHARED_TASK_NOTES  │
└─────────────────────────────────────────────────────┘
```

### Installation

> **Note:** ralph-copilot is the GitHub Actions equivalent of continuous-claude in this repo. See `ralph-copilot/` workflow files for the implementation.

### Usage

```bash
# Basic: 10 iterations via GitHub Actions workflow dispatch
gh workflow run ralph-copilot.yml \
  --field prompt="Add unit tests for all untested functions" \
  --field max_runs=10

# Cost-limited (via iteration count approximation)
gh workflow run ralph-copilot.yml \
  --field prompt="Fix all linter errors" \
  --field max_runs=20

# Time-boxed
gh workflow run ralph-copilot.yml \
  --field prompt="Improve test coverage" \
  --field max_duration=8h

# With code review pass
gh workflow run ralph-copilot.yml \
  --field prompt="Add authentication feature" \
  --field max_runs=10 \
  --field review_prompt="Run npm test && npm run lint, fix any failures"

# Parallel via worktrees
gh workflow run ralph-copilot.yml --field prompt="Add tests" --field max_runs=5 --field worktree=tests-worker
gh workflow run ralph-copilot.yml --field prompt="Refactor code" --field max_runs=5 --field worktree=refactor-worker
```

### Cross-Iteration Context: SHARED_TASK_NOTES.md

The critical innovation: a `SHARED_TASK_NOTES.md` file persists across iterations:

```markdown
## Progress
- [x] Added tests for auth module (iteration 1)
- [x] Fixed edge case in token refresh (iteration 2)
- [ ] Still need: rate limiting tests, error boundary tests

## Next Steps
- Focus on rate limiting module next
- The mock setup in tests/helpers.ts can be reused
```

Copilot reads this file at iteration start and updates it at iteration end. This bridges the context gap between independent Copilot agent sessions.

### CI Failure Recovery

When PR checks fail, the Ralph-Copilot loop automatically:
1. Fetches the failed run ID via `gh run list`
2. Creates a new GitHub Issue with CI fix context, assigns to `@copilot`
3. Copilot inspects logs via `gh run view`, fixes code, commits, pushes
4. Re-waits for checks (up to `max_ci_retries` attempts)

### Completion Signal

Copilot can signal "I'm done" by writing a magic phrase to `SHARED_TASK_NOTES.md`:

```bash
gh workflow run ralph-copilot.yml \
  --field prompt="Fix all bugs in the issue tracker" \
  --field completion_signal="RALPH_COPILOT_PROJECT_COMPLETE" \
  --field completion_threshold=3  # Stops after 3 consecutive signals
```

Three consecutive iterations signaling completion stops the loop, preventing wasted runs on finished work.

### Key Configuration

| Field | Purpose |
|------|---------|
| `max_runs` | Stop after N successful iterations |
| `max_duration` | Stop after time elapsed |
| `merge_strategy` | squash, merge, or rebase |
| `worktree` | Parallel execution via git worktrees |
| `disable_commits` | Dry-run mode (no git operations) |
| `review_prompt` | Add reviewer pass per iteration |
| `max_ci_retries` | Auto-fix CI failures (default: 1) |

---

## 5. The De-Sloppify Pattern

**An add-on pattern for any loop.** Add a dedicated cleanup/refactor step after each Implementer step.

### The Problem

When you ask an LLM to implement with TDD, it takes "write tests" too literally:
- Tests that verify TypeScript's type system works (testing `typeof x === 'string'`)
- Overly defensive runtime checks for things the type system already guarantees
- Tests for framework behavior rather than business logic
- Excessive error handling that obscures the actual code

### Why Not Negative Instructions?

Adding "don't test type systems" or "don't add unnecessary checks" to the Implementer prompt has downstream effects:
- The model becomes hesitant about ALL testing
- It skips legitimate edge case tests
- Quality degrades unpredictably

### The Solution: Separate Pass

Instead of constraining the Implementer, let it be thorough. Then add a focused cleanup agent:

```bash
# Step 1: Implement (let it be thorough)
gh issue create \
  --title "feat: implement the feature with full TDD" \
  --body "Implement the feature with full TDD. Be thorough with tests." \
  --assignee "@copilot" --label "copilot"

# Step 2: De-sloppify (separate Copilot session, focused cleanup)
gh issue create \
  --title "chore: de-sloppify implementation" \
  --body "Review all changes in the working tree. Remove:
- Tests that verify language/framework behavior rather than business logic
- Redundant type checks that the type system already enforces
- Over-defensive error handling for impossible states
- Console.log statements
- Commented-out code

Keep all business logic tests. Run the test suite after cleanup to ensure nothing breaks." \
  --assignee "@copilot" --label "copilot"
```

### In a Loop Context

```bash
for feature in "${features[@]}"; do
  # Implement
  gh issue create --title "feat: add $feature" \
    --body "Implement $feature with TDD." \
    --assignee "@copilot" --label "copilot"
  # (wait for PR merge here in practice)

  # De-sloppify
  gh issue create --title "chore: de-sloppify $feature" \
    --body "Cleanup pass: review changes, remove test/code slop, run tests." \
    --assignee "@copilot" --label "copilot"

  # Verify
  gh issue create --title "chore: verify $feature build" \
    --body "Run build + lint + tests. Fix any failures." \
    --assignee "@copilot" --label "copilot"
done
```

### Key Insight

> Rather than adding negative instructions which have downstream quality effects, add a separate de-sloppify pass. Two focused agents outperform one constrained agent.

---

## 6. Ralphinho / RFC-Driven DAG Orchestration

**The most sophisticated pattern.** An RFC-driven, multi-agent pipeline that decomposes a spec into a dependency DAG, runs each unit through a tiered quality pipeline, and lands them via an agent-driven merge queue. Created by enitrat (credit: @enitrat).

### Architecture Overview

```
RFC/PRD Document
       │
       ▼
  DECOMPOSITION (AI)
  Break RFC into work units with dependency DAG
       │
       ▼
┌──────────────────────────────────────────────────────┐
│  RALPH LOOP (up to 3 passes)                         │
│                                                      │
│  For each DAG layer (sequential, by dependency):     │
│                                                      │
│  ┌── Quality Pipelines (parallel per unit) ───────┐  │
│  │  Each unit in its own worktree:                │  │
│  │  Research → Plan → Implement → Test → Review   │  │
│  │  (depth varies by complexity tier)             │  │
│  └────────────────────────────────────────────────┘  │
│                                                      │
│  ┌── Merge Queue ─────────────────────────────────┐  │
│  │  Rebase onto main → Run tests → Land or evict │  │
│  │  Evicted units re-enter with conflict context  │  │
│  └────────────────────────────────────────────────┘  │
│                                                      │
└──────────────────────────────────────────────────────┘
```

### RFC Decomposition

AI reads the RFC and produces work units:

```typescript
interface WorkUnit {
  id: string;              // kebab-case identifier
  name: string;            // Human-readable name
  rfcSections: string[];   // Which RFC sections this addresses
  description: string;     // Detailed description
  deps: string[];          // Dependencies (other unit IDs)
  acceptance: string[];    // Concrete acceptance criteria
  tier: "trivial" | "small" | "medium" | "large";
}
```

**Decomposition Rules:**
- Prefer fewer, cohesive units (minimize merge risk)
- Minimize cross-unit file overlap (avoid conflicts)
- Keep tests WITH implementation (never separate "implement X" + "test X")
- Dependencies only where real code dependency exists

The dependency DAG determines execution order:
```
Layer 0: [unit-a, unit-b]     ← no deps, run in parallel
Layer 1: [unit-c]             ← depends on unit-a
Layer 2: [unit-d, unit-e]     ← depend on unit-c
```

### Complexity Tiers

Different tiers get different pipeline depths:

| Tier | Pipeline Stages |
|------|----------------|
| **trivial** | implement → test |
| **small** | implement → test → code-review |
| **medium** | research → plan → implement → test → PRD-review + code-review → review-fix |
| **large** | research → plan → implement → test → PRD-review + code-review → review-fix → final-review |

This prevents expensive operations on simple changes while ensuring architectural changes get thorough scrutiny.

### Separate Context Windows (Author-Bias Elimination)

Each stage runs in its own agent process with its own context window:

| Stage | Model | Purpose |
|-------|-------|---------|
| Research | Sonnet | Read codebase + RFC, produce context doc |
| Plan | Opus | Design implementation steps |
| Implement | Codex | Write code following the plan |
| Test | Sonnet | Run build + test suite |
| PRD Review | Sonnet | Spec compliance check |
| Code Review | Opus | Quality + security check |
| Review Fix | Codex | Address review issues |
| Final Review | Opus | Quality gate (large tier only) |

**Critical design:** The reviewer never wrote the code it reviews. This eliminates author bias — the most common source of missed issues in self-review.

### Merge Queue with Eviction

After quality pipelines complete, units enter the merge queue:

```
Unit branch
    │
    ├─ Rebase onto main
    │   └─ Conflict? → EVICT (capture conflict context)
    │
    ├─ Run build + tests
    │   └─ Fail? → EVICT (capture test output)
    │
    └─ Pass → Fast-forward main, push, delete branch
```

**File Overlap Intelligence:**
- Non-overlapping units land speculatively in parallel
- Overlapping units land one-by-one, rebasing each time

**Eviction Recovery:**
When evicted, full context is captured (conflicting files, diffs, test output) and fed back to the implementer on the next Ralph pass:

```markdown
## MERGE CONFLICT — RESOLVE BEFORE NEXT LANDING

Your previous implementation conflicted with another unit that landed first.
Restructure your changes to avoid the conflicting files/lines below.

{full eviction context with diffs}
```

### Data Flow Between Stages

```
research.contextFilePath ──────────────────→ plan
plan.implementationSteps ──────────────────→ implement
implement.{filesCreated, whatWasDone} ─────→ test, reviews
test.failingSummary ───────────────────────→ reviews, implement (next pass)
reviews.{feedback, issues} ────────────────→ review-fix → implement (next pass)
final-review.reasoning ────────────────────→ implement (next pass)
evictionContext ───────────────────────────→ implement (after merge conflict)
```

### Worktree Isolation

Every unit runs in an isolated worktree (uses jj/Jujutsu, not git):
```
/tmp/workflow-wt-{unit-id}/
```

Pipeline stages for the same unit **share** a worktree, preserving state (context files, plan files, code changes) across research → plan → implement → test → review.

### Key Design Principles

1. **Deterministic execution** — Upfront decomposition locks in parallelism and ordering
2. **Human review at leverage points** — The work plan is the single highest-leverage intervention point
3. **Separate concerns** — Each stage in a separate context window with a separate agent
4. **Conflict recovery with context** — Full eviction context enables intelligent re-runs, not blind retries
5. **Tier-driven depth** — Trivial changes skip research/review; large changes get maximum scrutiny
6. **Resumable workflows** — Full state persisted to SQLite; resume from any point

### When to Use Ralphinho vs Simpler Patterns

| Signal | Use Ralphinho | Use Simpler Pattern |
|--------|--------------|-------------------|
| Multiple interdependent work units | Yes | No |
| Need parallel implementation | Yes | No |
| Merge conflicts likely | Yes | No (sequential is fine) |
| Single-file change | No | Yes (sequential pipeline) |
| Multi-day project | Yes | Maybe (ralph-copilot) |
| Spec/RFC already written | Yes | Maybe |
| Quick iteration on one thing | No | Yes (Sequential Pipeline or Copilot Chat) |

---

## Choosing the Right Pattern

### Decision Matrix

```
Is the task a single focused change?
├─ Yes → Sequential Pipeline (GitHub Actions) or Copilot Chat
└─ No → Is there a written spec/RFC?
         ├─ Yes → Do you need parallel implementation?
         │        ├─ Yes → Ralphinho (DAG orchestration)
         │        └─ No → Ralph-Copilot Loop (iterative PR loop)
         └─ No → Do you need many variations of the same thing?
                  ├─ Yes → Infinite Agentic Loop (spec-driven generation)
                  └─ No → Sequential Pipeline with de-sloppify
```

### Combining Patterns

These patterns compose well:

1. **Sequential Pipeline + De-Sloppify** — The most common combination. Every implement step gets a cleanup pass.

2. **Ralph-Copilot + De-Sloppify** — Add a `review_prompt` with a de-sloppify directive to each iteration.

3. **Any loop + Verification** — Use ECC's `/verify` command or `verification-loop` skill as a gate before commits.

4. **Ralphinho's tiered approach in simpler loops** — Even in a sequential pipeline, you can route simple tasks vs complex tasks differently by specifying complexity in the issue body and having the agent decide scope:
   ```bash
   # Simple formatting fix — brief issue body
   gh issue create --title "fix: import ordering in src/utils.ts" \
     --body "Fix the import ordering in src/utils.ts. No other changes." \
     --assignee "@copilot" --label "copilot"

   # Complex architectural change — detailed issue body with full context
   gh issue create --title "refactor: auth module to strategy pattern" \
     --body "$(cat docs/auth-refactor-plan.md)" \
     --assignee "@copilot" --label "copilot"
   ```

---

## Anti-Patterns

### Common Mistakes

1. **Infinite loops without exit conditions** — Always have a max-runs, max-cost, max-duration, or completion signal.

2. **No context bridge between iterations** — Each Copilot agent session starts fresh. Use `SHARED_TASK_NOTES.md` or filesystem state to bridge context.

3. **Retrying the same failure** — If an iteration fails, don't just retry. Capture the error context and feed it to the next attempt.

4. **Negative instructions instead of cleanup passes** — Don't say "don't do X." Add a separate pass that removes X.

5. **All agents in one context window** — For complex workflows, separate concerns into different agent processes. The reviewer should never be the author.

6. **Ignoring file overlap in parallel work** — If two parallel agents might edit the same file, you need a merge strategy (sequential landing, rebase, or conflict resolution).

---

## References

| Project | Author | Link |
|---------|--------|------|
| Ralphinho | enitrat | credit: @enitrat |
| Infinite Agentic Loop | disler | credit: @disler |
| Continuous Claude (adapted as Ralph-Copilot) | AnandChowdhary | credit: @AnandChowdhary |
| NanoClaw | ECC | ⚠️ Not available in Copilot — Claude Code-specific |
| Verification Loop | ECC | `skills/verification-loop/` in this repo |
