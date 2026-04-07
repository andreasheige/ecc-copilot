---
applyTo: "**"
---

# Pipeline Artifact & Learning Protocol

Every agent in the pipeline MUST follow this protocol to collect artifacts and improve over time.

> **Important**: Subagents spawned via `runSubagent` do NOT automatically receive these instructions.
> The orchestrator (architect) MUST embed the artifact and logging requirements into every subagent prompt.
> See the "Subagent Prompt Template" section below for the required boilerplate.

## ⛔ Orchestrator Hard Rule (No Exceptions)

The architect agent MUST create the session folder and `00-scope.md` as its **very first tool calls** — before reading any source file, before writing any code, before dispatching any subagent.

**Enforcement checklist** (run mentally at start of every task):
- [ ] `mkdir -p .github/pipeline-artifacts/sessions/<YYYY-MM-DD-username-task-slug>/` — done?
- [ ] `00-scope.md` written? — done?
- [ ] `agent-log.jsonl` created with `start` entry? — done?
- [ ] `architecture.md` learnings read? — done?

If any box is unchecked → do it now before proceeding.

Skipping is a protocol violation. There are no "small task" exemptions.

## Two-Tier Storage Model (Multi-Developer)

This repo is used by 15+ developers simultaneously. Artifacts are split into two tiers to eliminate conflicts:

| Tier | Path | Git tracked? | Purpose |
|---|---|---|---|
| **Sessions** | `.github/pipeline-artifacts/sessions/` | ❌ `.gitignore`d | Per-developer, per-task scratch space. Never committed. |
| **Learnings** | `.github/pipeline-artifacts/learnings/` | ✅ Committed | Shared team knowledge base. Commit after each task. |

**Sessions are local-only** — they exist only on the developer's machine during and after the session. They are never pushed, never cause merge conflicts, and never pollute PRs.

**Learnings are shared** — after completing a task, the architect appends extracted insights to the relevant `learnings/*.md` file and includes that change in the PR.

### Session Folder Naming

To prevent the rare collision where two developers work on the same JIRA ticket on the same day, session folders MUST include the developer's git username:

```
.github/pipeline-artifacts/sessions/<YYYY-MM-DD>-<git-username>-<task-slug>/
```

**Example**: `2026-04-07-aheige-pbcde-15050-tealium-guest-bpid/`

Get the git username with: `git config user.name | tr ' ' '-' | tr '[:upper:]' '[:lower:]'`

## Directory Structure

```
.github/pipeline-artifacts/
  sessions/                           ← .gitignored (local only)
    <YYYY-MM-DD>-<user>-<task-slug>/  # One folder per developer per run
      00-scope.md                      # architect output
      01-analysis.md                   # analyser output
      01-work-items.md                 # work-item-creator output
      02-<agent-name>.md               # each dev agent's report
      03-code-quality.md               # review gate
      03-security.md                   # review gate
      04-qa-<type>.md                  # QA gate reports
      05-deploy.md                     # deploy report
  learnings/                          ← committed, shared across team
    code-quality.md                    # Patterns from code reviews
    security.md                        # Common security issues found
    architecture.md                    # Architectural decisions & patterns
    testing.md                         # Testing gaps & patterns
    performance.md                     # Performance patterns & budgets
    frontend.md                        # Frontend patterns & pitfalls
    backend.md                         # Backend patterns & pitfalls
    general.md                         # Cross-cutting learnings
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

## Subagent Prompt Template (MANDATORY for orchestrator)

When the orchestrator dispatches a subagent via `runSubagent`, it MUST append the following block to the prompt. Replace placeholders with actual values.

````
## Artifact Protocol (MANDATORY)

You are running as part of a pipeline. You MUST write artifacts before returning.

**Session folder**: `.github/pipeline-artifacts/sessions/{{SESSION_FOLDER}}/`
**Your artifact file**: `{{STAGE}}-{{AGENT_NAME}}.md`
**Log file**: `.github/pipeline-artifacts/sessions/{{SESSION_FOLDER}}/agent-log.jsonl`

### Step 1: Log your start
Append this line to the log file (create if it doesn't exist):
```json
{"event":"start","agent":"{{AGENT_NAME}}","stage":{{STAGE_NUMBER}},"model":"{{MODEL}}","timestamp":"{{ISO_TIMESTAMP}}","task":"{{TASK_SUMMARY}}"}
```

### Step 2: Do your work
Complete the task described above.

### Step 3: Write your artifact
Create the file `.github/pipeline-artifacts/sessions/{{SESSION_FOLDER}}/{{STAGE}}-{{AGENT_NAME}}.md` with this format:
```markdown
# {{AGENT_NAME}} — {{TASK_SUMMARY}}

**Session**: {{SESSION_FOLDER}}
**Date**: {{DATE}}
**Agent**: {{AGENT_NAME}}
**Stage**: {{STAGE_NUMBER}}
**Status**: PASS | FAIL | BLOCKED

## Summary
<2-3 sentence summary>

## Changes
- <file>: <what changed>

## Findings
- <issues, risks, decisions>
```

### Step 4: Log your end
Append this line to the log file:
```json
{"event":"end","agent":"{{AGENT_NAME}}","stage":{{STAGE_NUMBER}},"status":"PASS|FAIL|BLOCKED","timestamp":"{{ISO_TIMESTAMP}}","tool_calls":{{COUNT}},"findings":{{COUNT}}}
```

### Step 5: Return your status
Your final message MUST start with `STATUS: PASS`, `STATUS: FAIL`, or `STATUS: BLOCKED` so the orchestrator can parse your result programmatically.
````

### Orchestrator self-artifact checklist

Before dispatching any subagent, the orchestrator MUST have already:

1. Created the session folder (use `create_file` or `run_in_terminal mkdir -p`)
2. Written `00-scope.md` into the session folder
3. Appended its own `start` log entry to `agent-log.jsonl`

After all stages complete, the orchestrator MUST:

1. Append its own `end` log entry
2. Invoke `session-reporter` with the session folder path in the prompt
3. Print the session summary to the user

## Memory Storage Progression

The learning system evolves with project maturity. Every agent MUST check the current tier and act accordingly.

### How to detect the current tier

| Check | Result | Tier |
|-------|--------|------|
| `learnings/*.md` files exist and are < 200 lines each | Yes | **Tier 1 — Markdown** |
| Any `learnings/*.md` exceeds 200 lines | Yes | **Trigger Tier 2 migration** |
| `learnings/*.jsonl` files exist | Yes | **Tier 2 — JSONL** |
| Any `learnings/*.jsonl` exceeds 500 entries | Yes | **Trigger Tier 3 migration** |

### Tier 1 — Markdown (current)

- Store learnings as `### Title — YYYY-MM-DD` entries in `learnings/*.md`
- Agents read the full file on start
- Simple, diffable, reviewable in PRs

### Tier 2 — Structured JSONL

**Trigger**: Any `learnings/*.md` file exceeds 200 lines.

**Migration action** (performed by the agent that detects the threshold):
1. Convert entries to JSONL: `{"date":"YYYY-MM-DD","domain":"security","title":"...","body":"...","source_agent":"...","session":"..."}`
2. Write to `learnings/<domain>.jsonl` alongside the existing `.md`
3. Keep the `.md` as a human-readable summary (top 20 most impactful learnings)
4. Future agents write to `.jsonl` and update the `.md` summary only for high-severity items

**Read pattern**: Agents read the slim `.md` summary first. If the current task matches a domain, also grep the `.jsonl` for relevant keywords.

### Tier 3 — Semantic / Vector (future)

**Trigger**: Any `learnings/*.jsonl` exceeds 500 entries.

**Migration action**:
1. Propose a RAG-based retrieval skill in `.github/skills/learning-retrieval/`
2. Embed all JSONL entries into a vector store (technology TBD based on project stack)
3. Agents query by semantic similarity instead of reading full files
4. The `.md` summaries remain as fallback for agents without vector access

**Do NOT implement Tier 3 speculatively** — only when the 500-entry threshold is hit and an engineer confirms the approach.
