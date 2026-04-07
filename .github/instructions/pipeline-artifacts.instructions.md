---
applyTo: ".github/agents/**"
---

# Pipeline Artifact & Learning Protocol

Every agent in the pipeline MUST follow this protocol to collect artifacts and improve over time.

## Directory Structure

```
.github/pipeline-artifacts/
  sessions/
    <YYYY-MM-DD-task-slug>/     # One folder per pipeline run
      00-scope.md                # architect output
      01-analysis.md             # analyser output
      01-work-items.md           # work-item-creator output
      02-<agent-name>.md         # each dev agent's report
      03-code-quality.md         # review gate
      03-security.md             # review gate
      04-qa-<type>.md            # QA gate reports
      05-deploy.md               # deploy report
  learnings/
    code-quality.md              # Patterns from code reviews
    security.md                  # Common security issues found
    architecture.md              # Architectural decisions & patterns
    testing.md                   # Testing gaps & patterns
    performance.md               # Performance patterns & budgets
    frontend.md                  # Frontend patterns & pitfalls
    backend.md                   # Backend patterns & pitfalls
    general.md                   # Cross-cutting learnings
```

## Before Starting Work

1. Read `.github/pipeline-artifacts/learnings/<your-topic>.md` (if it exists)
2. Read the current session folder for any previous stage artifacts
3. Apply past learnings to your current task — avoid repeating known mistakes

## After Completing Work

1. **Write your artifact** to `.github/pipeline-artifacts/sessions/<session>/<stage>-<name>.md` using the format below
2. **Extract learnings** — if you discovered something reusable (a pattern, pitfall, or decision), append it to the relevant `learnings/*.md` file

## Artifact Format

```markdown
# <Agent Name> — <Task Summary>

**Session**: <session-folder-name>
**Date**: <YYYY-MM-DD>
**Agent**: <agent-name>
**Model**: <model used, e.g. "Claude Sonnet 4.5">
**Stage**: <0-5>
**Status**: PASS | FAIL | BLOCKED
**Started**: <HH:MM>
**Finished**: <HH:MM>
**Duration**: <e.g. "3 min" or "~45 sec">
**Tool calls**: <approximate count of tool invocations>

## Summary

<2-3 sentence summary of what was done>

## Changes

- <file>: <what changed and why>

## Tests

- <tests added/modified>
- Coverage: <before → after if applicable>

## Findings

- <anything notable: risks, tech debt, patterns, decisions>

## Learnings Extracted

- <new insight added to learnings/\*.md, or "none">
```

## Learning Entry Format

When appending to `learnings/*.md`:

```markdown
### <Short Title> — <YYYY-MM-DD>

<1-3 sentences describing the learning, why it matters, and what to do differently>
```

## Rules for Gate Agents (Review + QA)

- DO NOT edit source code — only write artifacts
- Your artifact MUST include a clear `**Status**: PASS` or `**Status**: FAIL`
- If FAIL: list each issue with severity and suggested fix
- If PASS: confirm what was validated

## Rules for the Orchestrator (architect)

- Create the session folder at pipeline start: `.github/pipeline-artifacts/sessions/<YYYY-MM-DD-task-slug>/`
- Write `00-scope.md` as your first artifact
- After pipeline completes, write a final summary appended to the session folder
