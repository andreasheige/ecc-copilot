---
applyTo: "**"
---

# Model Selection Matrix

Every agent and skill MUST select models based on the task type and the user's Copilot plan. Do NOT hardcode a single model — use the tier system below.

## Available Models by Plan

| Model | Free | Pro | Pro+ | Business | Enterprise |
|-------|------|-----|------|----------|------------|
| GPT-4.1 | ✅ | ✅ | ✅ | ✅ | ✅ |
| GPT-5 mini | ✅ | ✅ | ✅ | ✅ | ✅ |
| GPT-5.4 mini | — | ✅ | ✅ | ✅ | ✅ |
| Grok Code Fast 1 | ✅ | ✅ | ✅ | ✅ | ✅ |
| Claude Haiku 4.5 | ✅ | ✅ | ✅ | ✅ | ✅ |
| Gemini 3 Flash | — | ✅ | ✅ | ✅ | ✅ |
| Claude Sonnet 4 | — | — | ✅ | ✅ | ✅ |
| Claude Sonnet 4.5 | — | — | ✅ | ✅ | ✅ |
| Claude Sonnet 4.6 | — | — | ✅ | ✅ | ✅ |
| GPT-5.2 | — | ✅ | ✅ | ✅ | ✅ |
| GPT-5.3-Codex | — | ✅ | ✅ | ✅ | ✅ |
| GPT-5.4 | — | — | ✅ | ✅ | ✅ |
| Gemini 2.5 Pro | — | ✅ | ✅ | ✅ | ✅ |
| Gemini 3.1 Pro | — | ✅ | ✅ | ✅ | ✅ |
| Claude Opus 4.5 | — | — | ✅ | ✅ | ✅ |
| Claude Opus 4.6 | — | — | ✅ | ✅ | ✅ |

## Premium Request Multipliers

| Cost Tier | Multiplier | Models |
|-----------|-----------|--------|
| **Free** (0x) | 0 | GPT-4.1, GPT-5 mini, Raptor mini |
| **Budget** (0.25x) | 0.25 | Grok Code Fast 1 |
| **Economy** (0.33x) | 0.33 | Claude Haiku 4.5, Gemini 3 Flash, GPT-5.4 mini |
| **Standard** (1x) | 1 | Claude Sonnet 4/4.5/4.6, GPT-5.2, GPT-5.3-Codex, GPT-5.4, Gemini 2.5 Pro, Gemini 3.1 Pro |
| **Premium** (3x) | 3 | Claude Opus 4.5, Claude Opus 4.6 |

## Model Selection by Task Type

Use this matrix to pick the right model for the job:

| Task Type | Recommended Model | Fallback | Cost Tier | Rationale |
|-----------|------------------|----------|-----------|-----------|
| **Orchestration / Planning** | Claude Opus 4.6 | GPT-5.4 | Premium | Complex multi-step reasoning, full pipeline coordination |
| **Code Generation** | Claude Sonnet 4.5 | GPT-5.3-Codex | Standard | Best code quality, strong reasoning |
| **Code Review** | *Multi-model parallel* | — | Mixed | See "Multi-Model Review Protocol" below |
| **Quick Fixes / Build Errors** | Claude Haiku 4.5 | GPT-5 mini | Economy/Free | Speed over depth, simple targeted fixes |
| **Test Writing** | Claude Sonnet 4 | GPT-5.2 | Standard | Good code gen, cost-efficient for volume |
| **Documentation** | GPT-5.4 mini | GPT-5 mini | Economy/Free | Natural language strength, low cost |
| **Security Review** | Claude Opus 4.6 | Claude Sonnet 4.5 | Premium/Standard | Deep reasoning needed for vuln detection |
| **Data Analysis / Search** | Gemini 3.1 Pro | Gemini 2.5 Pro | Standard | Strong analytical and multimodal |
| **Fast Triage / Filtering** | Grok Code Fast 1 | GPT-4.1 | Budget/Free | Fastest response, lowest cost |

## Multi-Model Review Protocol

Code reviews MUST use parallel agents with different models for diversity of perspective.

### Rules

- **Small PR** (≤ 10 changed lines): Single reviewer is sufficient
- **Normal PR** (11–500 changed lines): Minimum 3 parallel reviewers, different models
- **Large PR** (> 500 changed lines): Maximum 5 parallel reviewers, different models

### Recommended Review Panel

| Panel Size | Models (pick from, ensure provider diversity) |
|-----------|-----------------------------------------------|
| 3 reviewers | Claude Sonnet 4.5, GPT-5.3-Codex, Gemini 3.1 Pro |
| 4 reviewers | Claude Sonnet 4.5, GPT-5.3-Codex, Gemini 3.1 Pro, Claude Opus 4.6 |
| 5 reviewers | Claude Sonnet 4.5, GPT-5.3-Codex, Gemini 3.1 Pro, Claude Opus 4.6, GPT-5.4 |

### Why Multi-Model?

- Different models catch different categories of issues
- Provider diversity eliminates shared blind spots
- Consensus across models = high confidence finding
- Disagreement between models = worth human attention

### Review Aggregation

After all parallel reviews complete:
1. **Unanimous findings** → auto-flag as confirmed issues
2. **Majority findings** (2+ models agree) → likely real, prioritize
3. **Single-model findings** → note but don't auto-flag, may be false positive
4. **Produce a merged report** with finding source attribution

## Budget-Conscious Mode

For teams watching premium request usage:

| Instead of | Use | Saves |
|-----------|-----|-------|
| Opus for orchestration | Sonnet 4.5 | 2x per call |
| 5 parallel reviewers | 3 parallel reviewers | 2 review calls |
| Sonnet for docs | GPT-5 mini (free) | 1x per call |
| Sonnet for build fixes | Haiku 4.5 (0.33x) | 0.67x per call |

## How Agents Should Reference This

In agent frontmatter, use the **task-appropriate** model:

```yaml
# Orchestrator — needs deep reasoning
model: "Claude Opus 4.6"

# Standard dev work — good quality, 1x cost
model: ["Claude Sonnet 4.5", "Claude Sonnet 4"]

# Fast/cheap tasks — build fixes, docs, triage
model: ["Claude Haiku 4.5", "GPT-5 mini"]

# Multi-model review — see Multi-Model Review Protocol
# (configured in agent body, not frontmatter)
```

## Plan Detection

Agents cannot directly detect the user's plan. Instead:
1. Try the recommended model
2. If the model is unavailable (error/fallback), drop to the next tier
3. Log which model was actually used in the agent-log.jsonl
