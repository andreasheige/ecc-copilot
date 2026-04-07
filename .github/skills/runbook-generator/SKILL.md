---
name: runbook-generator
description: >-
  Use when the project lacks debugging runbooks, after a production
  incident, or when onboarding new team members who need to understand
  how to diagnose issues. Scans the codebase for error handling, logging,
  infrastructure config, and common failure patterns, then generates
  project-specific debugging runbooks. DO NOT USE if runbooks already
  exist and are up to date.
---

# Runbook Generator

Generate project-specific debugging runbooks by analyzing the codebase's error handling, logging, and infrastructure.

## When to Use

- After a production incident (capture the investigation steps as a runbook)
- When onboarding new engineers to on-call
- When adding new services or external dependencies
- When the existing runbook doesn't cover a failure mode

## Workflow

### Step 1: Discover Error Surface

Scan the codebase for:

```
1. Error handling patterns:
   - try/catch blocks and what they catch
   - Error classes and custom error types
   - HTTP error responses and their codes
   - Error boundary components (React)

2. Logging setup:
   - Logger library (winston, pino, structlog, slog, etc.)
   - Log levels used and where
   - Structured fields (request IDs, user IDs, etc.)

3. External dependencies:
   - Database connections and health checks
   - API clients and their failure modes
   - Message queues, caches, CDNs
   - Third-party services (Stripe, Auth0, SendGrid, etc.)

4. Infrastructure:
   - Docker/K8s configs
   - CI/CD pipelines
   - Environment variables and secrets
   - Health check endpoints
```

### Step 2: Map Symptoms to Causes

For each external dependency and error class, document:

| Symptom | Likely Cause | Investigation Steps | Resolution |
|---------|-------------|--------------------:|------------|
| 503 from `/api/health` | DB connection pool exhausted | Check `pg_stat_activity`, review connection count | Restart pods, increase pool size |
| Timeout on search | Redis unavailable | Check Redis health endpoint, review memory usage | Failover to DB fallback |

### Step 3: Generate Runbook Skills

Create skills in `.github/skills/`:

```
.github/skills/runbook-<service>/
  SKILL.md              # Symptom → investigation → resolution
  queries/
    db-diagnostics.sql  # Common diagnostic queries
    log-patterns.md     # Grep patterns for common errors
  checklists/
    incident-response.md  # Step-by-step incident checklist
```

### Runbook SKILL.md Structure

```markdown
# Runbook: <Service Name>

## Quick Diagnosis

| Symptom | First Check | Escalation |
|---------|------------|------------|
| ... | ... | ... |

## Service Dependencies

<diagram or list of what this service depends on>

## Common Issues

### Issue 1: <Name>
**Symptom**: What the user/alert reports
**Root cause**: Why it happens
**Investigation**:
1. Check <specific thing>
2. Run <specific command>
3. Look for <specific pattern>
**Resolution**: How to fix it
**Prevention**: How to prevent recurrence

## Useful Commands

### Log Queries
<grepping patterns for this service's logs>

### Database Diagnostics
<queries to diagnose DB issues>

### Health Checks
<curl commands for health endpoints>
```

### Step 4: Validate

1. Does each symptom-to-resolution path actually work?
2. Are the commands/queries correct for the project's stack?
3. Is the runbook discoverable? (trigger-oriented description)

## Output

```markdown
## Runbooks Generated

| Runbook | Service | Issues Covered | Based On |
|---------|---------|---------------|----------|
| `runbook-api` | REST API | 5 error paths | error handlers in src/api/ |
| `runbook-auth` | Auth service | 3 failure modes | auth middleware, token refresh |
| `runbook-search` | Search/Redis | 4 scenarios | Redis client, vector search |
```

## Gotchas

- **Runbooks rot fast** — regenerate after major architecture changes
- **Don't guess failure modes** — only document issues that have actually happened or are clearly possible from the code
- **Include the boring stuff** — "restart the pod" is a valid resolution; not everything needs a complex fix
- **Link to monitoring** — reference actual dashboard URLs, alert names, and log queries
- **Test the commands** — a runbook with wrong commands is worse than no runbook
