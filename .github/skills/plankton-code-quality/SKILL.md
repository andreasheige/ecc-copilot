---
name: plankton-code-quality
description: >-
  Use when setting up or configuring Plankton code quality hooks
  (auto-formatting, linting, AI-powered fixes), debugging hook failures,
  resolving conflicts between Plankton and other formatters, or protecting
  linter configs from rule-gaming. DO NOT USE for manual code review (use
  code-quality-reviewer agent) or test coverage (use tdd-workflow).
---

# Plankton Code Quality Skill

Integration reference for Plankton (credit: @alxfazio), a write-time code quality enforcement system for GitHub Copilot. Plankton runs formatters and linters on every file edit via git hooks (lefthook equivalent), then runs linter commands directly to fix violations the agent didn't catch.

## Setup

Check `config.json` for project-specific settings (hook runner, formatter, linter choices, protected configs). If `config.json` is missing, ask the user which tools their project uses before configuring hooks.

## When to Use

- You want automatic formatting and linting on every file edit (not just at commit time)
- You need defense against agents modifying linter configs to pass instead of fixing code
- You want tiered model routing for fixes (Haiku for simple style, Sonnet for logic, Opus for types)
- You work with multiple languages (Python, TypeScript, Shell, YAML, JSON, TOML, Markdown, Dockerfile)

## How It Works

### Three-Phase Architecture

Every time GitHub Copilot edits or writes a file, Plankton's `multi_linter.sh` git hook (via lefthook) runs:

```
Phase 1: Auto-Format (Silent)
├─ Runs formatters (ruff format, biome, shfmt, taplo, markdownlint)
├─ Fixes 40-50% of issues silently
└─ No output to main agent

Phase 2: Collect Violations (JSON)
├─ Runs linters and collects unfixable violations
├─ Returns structured JSON: {line, column, code, message, linter}
└─ Still no output to main agent

Phase 3: Delegate + Verify
├─ Runs linter commands directly with violations JSON
├─ Routes to model tier based on violation complexity:
│   ├─ Fast model: formatting, imports, style (E/W/F codes) — 120s timeout
│   ├─ Standard model: complexity, refactoring (C901, PLR codes) — 300s timeout
│   └─ Strong model: type system, deep reasoning (unresolved-attribute) — 600s timeout
├─ Re-runs Phase 1+2 to verify fixes
└─ Exit 0 if clean, Exit 2 if violations remain (reported to main agent)
```

### What the Main Agent Sees

| Scenario | Agent sees | Hook exit |
|----------|-----------|-----------|
| No violations | Nothing | 0 |
| All fixed by subprocess | Nothing | 0 |
| Violations remain after subprocess | `[hook] N violation(s) remain` | 2 |
| Advisory (duplicates, old tooling) | `[hook:advisory] ...` | 0 |

The main agent only sees issues the subprocess couldn't fix. Most quality problems are resolved transparently.

### Config Protection (Defense Against Rule-Gaming)

LLMs will modify `.ruff.toml` or `biome.json` to disable rules rather than fix code. Plankton blocks this with three layers:

1. **Pre-commit hook** — `protect_linter_configs.sh` (via lefthook) blocks edits to all linter configs before they happen
2. **Post-commit hook** — `stop_config_guardian.sh` detects config changes via `git diff` at commit time
3. **Protected files list** — `.ruff.toml`, `biome.json`, `.shellcheckrc`, `.yamllint`, `.hadolint.yaml`, and more

### Package Manager Enforcement

A pre-commit hook (via lefthook) on Bash blocks legacy package managers:
- `pip`, `pip3`, `poetry`, `pipenv` → Blocked (use `uv`)
- `npm`, `yarn`, `pnpm` → Blocked (use `bun`)
- Allowed exceptions: `npm audit`, `npm view`, `npm publish`

## Setup

### Quick Start

> **Note:** Plankton requires manual installation from its repository. Review the code before installing.

```bash
# Install core dependencies
brew install jaq ruff uv

# Install Python linters
uv sync --all-extras

# Hooks activate automatically via lefthook on git operations
lefthook install
```

No separate Claude Code install needed. The hooks in `lefthook.yml` are picked up automatically when you run `lefthook install` in the Plankton directory.

### Per-Project Integration

To use Plankton hooks in your own project:

1. Copy the hook scripts directory to your project
2. Add hook configuration to `lefthook.yml` at repo root
3. Copy linter config files (`.ruff.toml`, `biome.json`, etc.)
4. Install the linters for your languages
5. Run `lefthook install` to activate

> ⚠️ **Note:** Plankton was originally designed for Claude Code's `.claude/hooks/` and `.claude/settings.json` system. In Copilot, these are replaced by `lefthook.yml` git hooks at the repo root.

### Language-Specific Dependencies

| Language | Required | Optional |
|----------|----------|----------|
| Python | `ruff`, `uv` | `ty` (types), `vulture` (dead code), `bandit` (security) |
| TypeScript/JS | `biome` | `oxlint`, `semgrep`, `knip` (dead exports) |
| Shell | `shellcheck`, `shfmt` | — |
| YAML | `yamllint` | — |
| Markdown | `markdownlint-cli2` | — |
| Dockerfile | `hadolint` (>= 2.12.0) | — |
| TOML | `taplo` | — |
| JSON | `jaq` | — |

## Pairing with ECC / APM

### Complementary, Not Overlapping

| Concern | APM (ECC port) | Plankton |
|---------|----------------|----------|
| Code quality enforcement | git hooks via lefthook (Prettier, tsc) | git hooks via lefthook (20+ linters + auto-fix) |
| Security scanning | AgentShield, security-reviewer agent | Bandit (Python), Semgrep (TypeScript) |
| Config protection | — | Pre-commit blocks + post-commit detection |
| Package manager | Detection + setup | Enforcement (blocks legacy PMs) |
| CI integration | — | Pre-commit hooks for git |
| Model routing | Manual (specify in prompt) | Automatic (violation complexity → tier) |

### Recommended Combination

1. Install APM as your Copilot instruction set (agents, skills, rules)
2. Add Plankton hooks for write-time quality enforcement (via lefthook)
3. Use the security-reviewer agent for security audits
4. Use APM's verification-loop as a final gate before PRs

### Avoiding Hook Conflicts

If running both ECC and Plankton hooks:
- ECC's Prettier hook and Plankton's biome formatter may conflict on JS/TS files
- Resolution: disable the APM Prettier git hook when using Plankton (Plankton's biome is more comprehensive)
- Both can coexist on different file types (ECC handles what Plankton doesn't cover)

## Configuration Reference

Plankton's `lefthook.yml` (or equivalent `config.json`) controls all behavior:

```json
{
  "languages": {
    "python": true,
    "shell": true,
    "yaml": true,
    "json": true,
    "toml": true,
    "dockerfile": true,
    "markdown": true,
    "typescript": {
      "enabled": true,
      "js_runtime": "auto",
      "biome_nursery": "warn",
      "semgrep": true
    }
  },
  "phases": {
    "auto_format": true,
    "subprocess_delegation": true
  },
  "subprocess": {
    "tiers": {
      "haiku":  { "timeout": 120, "max_turns": 10 },
      "sonnet": { "timeout": 300, "max_turns": 10 },
      "opus":   { "timeout": 600, "max_turns": 15 }
    },
    "volume_threshold": 5
  }
}
```

**Key settings:**
- Disable languages you don't use to speed up hooks
- `volume_threshold` — violations > this count auto-escalate to a higher model tier
- `subprocess_delegation: false` — skip Phase 3 entirely (just report violations)

## Environment Overrides

| Variable | Purpose |
|----------|---------|
| `HOOK_SKIP_SUBPROCESS=1` | Skip Phase 3, report violations directly |
| `HOOK_SUBPROCESS_TIMEOUT=N` | Override tier timeout |
| `HOOK_DEBUG_MODEL=1` | Log model selection decisions |
| `HOOK_SKIP_PM=1` | Bypass package manager enforcement |

## References

- Plankton (credit: @alxfazio)
- Plankton REFERENCE.md — Full architecture documentation (credit: @alxfazio)
- Plankton SETUP.md — Detailed installation guide (credit: @alxfazio)

## ECC v1.8 Additions

### Copyable Hook Profile

Set strict quality behavior:

```bash
export ECC_HOOK_PROFILE=strict
export ECC_QUALITY_GATE_FIX=true
export ECC_QUALITY_GATE_STRICT=true
```

### Language Gate Table

- TypeScript/JavaScript: Biome preferred, Prettier fallback
- Python: Ruff format/check
- Go: gofmt

### Config Tamper Guard

During quality enforcement, flag changes to config files in same iteration:

- `biome.json`, `.eslintrc*`, `prettier.config*`, `tsconfig.json`, `pyproject.toml`

If config is changed to suppress violations, require explicit review before merge.

### CI Integration Pattern

Use the same commands in CI as local hooks:

1. run formatter checks
2. run lint/type checks
3. fail fast on strict mode
4. publish remediation summary

### Health Metrics

Track:
- edits flagged by gates
- average remediation time
- repeat violations by category
- merge blocks due to gate failures
