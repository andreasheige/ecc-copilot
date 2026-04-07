---
applyTo: "**"
---

# ECC for Copilot — Core Behavior

> This instruction set ports Everything Claude Code (ECC) v1.9.0 patterns to GitHub Copilot.
> It replicates the proactive, agent-first, TDD-first behavior of Claude Code.

---

## 1. Agent-First Behavior (Proactive — No Prompting Required)

Spawn specialized agents **automatically** based on context. Do not wait to be asked.

| When this happens | Auto-invoke this agent |
|-------------------|----------------------|
| Complex feature request (>1 file) | `planner` — plan before writing any code |
| Code just written or modified | `code-reviewer` — review immediately |
| New feature or bug fix | `tdd-guide` — TDD workflow |
| Architectural decision needed | `architect` — design first |
| Security-sensitive code (auth, payments, PII) | `security-reviewer` — mandatory |
| Build or type error | `build-error-resolver` — fix before continuing |
| Dead code / large refactor | `refactor-cleaner` |
| Documentation needs updating | `doc-updater` |

**How to invoke:** Use `/agent <name>` or ask Copilot to "act as the planner agent" for that session.

---

## 2. TDD Enforcement (Non-Negotiable)

**Every non-trivial code change follows Red → Green → Refactor.**

- "Implement X" → Load `tdd-workflow` skill → write tests first → implement → refactor
- "Fix bug in X" → Write a failing test that reproduces the bug → fix → verify
- "Refactor X" → Lock tests first → refactor → all tests must still pass
- Never write implementation before a failing test exists

**Verification gate** — before any commit, run the `verification-loop` skill:
1. Build passes
2. TypeScript/types check clean
3. Lint passes with zero warnings
4. Tests pass with 80%+ coverage

---

## 3. Skills Library (Load on Demand)

Skills live in `.github/skills/`. Reference them by saying "load the `<skill>` skill" or Copilot will load relevant ones automatically based on task type.

| Task type | Skill to load |
|-----------|--------------|
| New feature planning | `blueprint` |
| TDD workflow | `tdd-workflow` |
| API design | `api-design` |
| Architecture decisions | `architecture-decision-records` |
| Code quality enforcement | `plankton-code-quality` |
| Autonomous/loop work | `continuous-agent-loop` |
| Large RFC feature | `ralphinho-rfc-pipeline` |
| Git workflow / PR | `git-workflow` |
| Pre-PR quality gate | `verification-loop` |
| AI-native engineering | `agentic-engineering` |

---

## 4. Security Rules (Always Active)

**Before ANY commit, verify:**
- No hardcoded secrets (API keys, passwords, tokens)
- All user inputs validated at boundaries
- SQL queries parameterized (never string-concatenated)
- HTML output sanitized (no XSS)
- Auth/authorization checked on every protected route
- Rate limiting on public endpoints
- Error messages don't expose stack traces or internals

**If security issue found:** STOP → invoke `security-reviewer` agent → fix before continuing.

**Mandatory — invoke `security-reviewer` agent for:**
- Any authentication or authorization code
- Payment / financial logic
- PII handling
- New API endpoints exposed to the internet

---

## 5. Code Quality Standards

### Immutability (Critical)
Always create new objects. Never mutate existing state.
```typescript
// ✅ Correct
const updated = { ...existing, field: newValue };
// ❌ Wrong
existing.field = newValue;
```

### File & Function Size
- Functions: < 50 lines
- Files: < 800 lines (200–400 ideal)
- Max nesting depth: 4 levels
- One concept per file

### Error Handling
- Handle errors explicitly at every level — never swallow silently
- User-facing: friendly, actionable messages
- Server-side: structured logging with context
- Never use bare `catch (e) {}` blocks

### Typing
- No `any` without explicit justification comment
- No `@ts-ignore` without explanation
- Prefer `unknown` over `any` for external data

---

## 6. Planning Before Execution

For any task touching more than 2 files or taking more than 30 minutes:
1. **Stop** — do not write code immediately
2. Invoke `planner` agent or load `blueprint` skill
3. Write a spec with: acceptance criteria, file list, dependency order, risks
4. Get confirmation before implementing
5. Execute in dependency order (no orphaned changes)

For large features (RFC-level):
- Load `ralphinho-rfc-pipeline` skill
- Decompose into work units with a DAG
- Implement one unit at a time, land each before starting the next

---

## 7. Mode Usage (Copilot-Specific)

| Mode | When to use |
|------|------------|
| **Interactive** (default) | Normal conversation + coding |
| **Plan** (`Shift+Tab`) | Review the plan before ANY code is written |
| **Autopilot** (`Shift+Tab` twice) | Fully autonomous execution of a defined task |

**Rule:** Always switch to Plan mode for tasks involving >2 files. Review the plan, then switch to Autopilot or Interactive to execute.

**`/fleet`** — use for parallel independent tasks (multiple agents working simultaneously).
**`/delegate`** — use to hand a well-defined task to the GitHub Copilot coding agent for a full PR.
**`/research`** — use before implementing anything involving external APIs or libraries.

---

## 8. Session Habits

| When | Action |
|------|--------|
| Starting a session | `/resume` — reload prior context |
| Context getting heavy | `/compact` — summarize and continue |
| End of session | `/share` — save transcript → extract learnings → add to instructions |
| After writing code | `/review` — always review before committing |
| Before complex work | `/plan` — structure before executing |

---

## 9. What Copilot Must Not Generate

**Code:**
- `any` without justification comment
- `// @ts-ignore` without explanation
- `console.log` in production code (use structured logger)
- Inline styles in React (use Tailwind/CSS modules)
- Class components (use functional)
- Hardcoded secrets

**Testing:**
- Implementation before tests (for non-trivial code)
- Tests that mock everything (they test nothing)
- Tests without assertions
- Snapshot tests as primary strategy

**Security:**
- `eval()` or `new Function()`
- `dangerouslySetInnerHTML` without sanitization
- `http://` URLs in production
- String-concatenated SQL queries

---

## 10. Parallel Execution

When tasks are independent, run them in parallel:

```
# Copilot CLI
/fleet → describe parallel tasks → Copilot spawns subagents

# Multiple worktrees for large parallel work
git worktree add ../feature-a -b feature/a
git worktree add ../feature-b -b feature/b
# Open separate Copilot sessions in each
```

Do not serialize work that can be parallelized. Identify independent tasks upfront (part of planning).
