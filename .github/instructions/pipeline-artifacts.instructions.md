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

## Invocation Logging

Every agent MUST log its own invocation by appending lines to the session log file. This is the Copilot equivalent of Claude Code's `PreToolUse` hooks — since Copilot has no native hook system, agents self-report.

**Log file**: `.github/pipeline-artifacts/sessions/<session>/agent-log.jsonl`

### When starting work — append:

```json
{"event":"start","agent":"<your-name>","stage":<0-6>,"model":"<model>","timestamp":"<ISO-8601>","task":"<1-line summary>"}
```

### When finishing work — append:

```json
{"event":"end","agent":"<your-name>","stage":<0-6>,"status":"PASS|FAIL|BLOCKED","timestamp":"<ISO-8601>","tool_calls":<count>,"findings":<count>}
```

### Rules

- Use `edit` tool to append (not overwrite) to `agent-log.jsonl`
- Timestamps in UTC ISO-8601: `2026-04-07T14:32:00Z`
- `tool_calls` = approximate number of tool invocations during your run
- `findings` = number of issues/items found (0 if not applicable)
- The log is the **primary data source** for `session-reporter` — if you don't log, you won't appear in the summary

### Example session log

```jsonl
{"event":"start","agent":"architect","stage":0,"model":"Claude Opus 4.6","timestamp":"2026-04-07T14:00:00Z","task":"Implement user profile page"}
{"event":"end","agent":"architect","stage":0,"status":"PASS","timestamp":"2026-04-07T14:02:30Z","tool_calls":12,"findings":0}
{"event":"start","agent":"analyser","stage":1,"model":"Claude Sonnet 4.5","timestamp":"2026-04-07T14:02:31Z","task":"Impact analysis for user profile"}
{"event":"end","agent":"analyser","stage":1,"status":"PASS","timestamp":"2026-04-07T14:03:45Z","tool_calls":8,"findings":3}
{"event":"start","agent":"frontend","stage":2,"model":"Claude Sonnet 4.5","timestamp":"2026-04-07T14:03:46Z","task":"Build ProfilePage component"}
{"event":"start","agent":"backend","stage":2,"model":"Claude Sonnet 4.5","timestamp":"2026-04-07T14:03:46Z","task":"Add /api/profile endpoint"}
```

---

## Before Starting Work

1. **Log your start** — append a `start` event to `agent-log.jsonl`
2. Read `.github/pipeline-artifacts/learnings/<your-topic>.md` (if it exists)
3. Read the current session folder for any previous stage artifacts
4. Apply past learnings to your current task — avoid repeating known mistakes

## After Completing Work

1. **Write your artifact** to `.github/pipeline-artifacts/sessions/<session>/<stage>-<name>.md` using the format below
2. **Extract learnings** — if you discovered something reusable (a pattern, pitfall, or decision), append it to the relevant `learnings/*.md` file
3. **Log your end** — append an `end` event to `agent-log.jsonl` with your status and tool call count

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
