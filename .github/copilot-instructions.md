# ECC Copilot — Global Instructions

> Powered by the ecc-copilot APM package (ECC v1.9.0 ported to GitHub Copilot).

This project uses a full multi-stage agent pipeline. See `.github/agents/` for all agents and `.github/skills/` for all workflow skills.

## Quick Reference — Agent Pipeline

| Stage | Agents | Trigger |
|-------|--------|---------|
| 0 — Orchestration | `architect` | Every non-trivial task |
| 1 — Analysis | `analyser`, `work-item-creator` | After architect scopes the work |
| 2 — Development (parallel) | `frontend`, `backend`, `api-expert`, `database`, `ai-ml`, `payments`, `rag-embedding`, `devsecops`, `dependency-supply-chain`, `documentation`, `test-automation`, `search-discovery`, `localization`, `analytics-data-layer`, `compliance-gdpr`, `notification-comms` | Parallel after analysis |
| 3 — Review (both must PASS) | `code-quality-reviewer`, `security-reviewer` | After all dev agents complete |
| 4 — QA (all must PASS) | `qa-functional`, `qa-integration-e2e`, `qa-performance`, `qa-automation-runner`, `contract-testing` | After review gates pass |
| 5 — Deploy | `devsecops-deploy` | After all QA gates pass |

## Core Behavioral Rules

See `.github/instructions/ecc-core.instructions.md` for the full behavioral ruleset. Summary:

- **Agent-first**: spawn specialized agents automatically
- **TDD-first**: write tests before implementation, always
- **Plan before code**: use `planner` or blueprint skill for >2-file changes
- **Security mandatory**: invoke `security-reviewer` for auth/payments/PII/endpoints
- **Verify before PR**: run `verification-loop` skill before every PR

## Session Habits

| When | Action |
|------|--------|
| Session start | `/resume` |
| Complex work | `/plan` or Shift+Tab → Plan mode |
| After code | `/review` |
| Context heavy | `/compact` |
| Session end | `/share` → extract learnings → update instructions |

## Skills Library

Load by name: "load the `<skill>` skill"

| Task | Skill |
|------|-------|
| Feature planning | `blueprint` |
| TDD workflow | `tdd-workflow` |
| API design | `api-design` |
| Architecture decisions | `architecture-decision-records` |
| Code quality | `plankton-code-quality` |
| Loop/autonomous work | `continuous-agent-loop` |
| Large RFC feature | `ralphinho-rfc-pipeline` |
| Git/PR workflow | `git-workflow` |
| Pre-PR quality gate | `verification-loop` |
| AI-native engineering | `agentic-engineering` |
