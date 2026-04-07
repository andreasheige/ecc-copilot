---
name: scaffold-generator
description: >-
  Use when the project lacks scaffolding skills for its stack, or when a
  user says "create a new component/endpoint/migration/service." Scans
  the codebase to detect frameworks, conventions, and patterns, then
  generates project-specific scaffolding skills with templates. DO NOT
  USE if scaffolding skills already exist for the detected stack.
---

# Scaffold Generator

Generate project-specific scaffolding skills by analyzing the actual codebase.

## When to Use

- User asks to scaffold something but no scaffolding skill exists yet
- After initial project setup, to bootstrap commonly needed generators
- When the team adopts a new framework or pattern

## Workflow

### Step 1: Detect Stack

Scan the project to identify:

```
- Package files: package.json, requirements.txt, go.mod, Cargo.toml, etc.
- Frameworks: React, Next.js, Vue, Angular, Django, FastAPI, Express, etc.
- Test frameworks: Jest, Vitest, Playwright, pytest, etc.
- CSS strategy: Tailwind, CSS Modules, styled-components, etc.
- State management: Redux, Zustand, Pinia, etc.
- ORM/DB: Prisma, Drizzle, SQLAlchemy, TypeORM, etc.
```

### Step 2: Analyze Conventions

For each detected pattern, find 2-3 existing examples in the codebase:

- **Components**: How are they structured? (file naming, exports, props typing)
- **API routes**: What's the pattern? (handler structure, validation, middleware)
- **Tests**: Where do they live? (co-located, `__tests__/`, `e2e/`)
- **Migrations**: Naming convention? (timestamps, sequential IDs)
- **Services**: How are they organized? (class-based, functional, DI pattern)

### Step 3: Generate Scaffolding Skills

For each detected opportunity, create a skill in `.github/skills/`:

```
.github/skills/new-<thing>/
  SKILL.md           # When to use, what it generates, gotchas
  templates/
    <file>.template   # Template files derived from actual codebase patterns
  config.json         # Project-specific settings (optional)
```

### Template Derivation Rules

- **DO**: Extract the pattern from existing code in the project
- **DO**: Preserve the project's naming conventions, import style, and file structure
- **DO**: Include the project's test pattern alongside the implementation template
- **DON'T**: Use generic patterns from training data
- **DON'T**: Add libraries or patterns not already in the project

### Step 4: Validate

For each generated skill:
1. Does the template match the project's conventions? (compare with 2-3 existing files)
2. Does the generated output pass lint and typecheck?
3. Is the description trigger-oriented? (says when to use, not what it is)

## Output

After generation, report:

```markdown
## Scaffolding Skills Generated

| Skill | Stack | Templates | Based On |
|-------|-------|-----------|----------|
| `new-component` | React + Tailwind | component.tsx, component.test.tsx | src/components/Button/ |
| `new-api-route` | Next.js | route.ts, route.test.ts | src/app/api/users/ |
| `new-migration` | Prisma | migration.sql | prisma/migrations/ |
```

## Gotchas

- **Don't generate skills for stacks you didn't detect** — if there's no ORM, don't create `new-migration`
- **Re-run when conventions change** — if the team refactors their component pattern, the scaffolding skill needs updating
- **Templates are starting points** — they should be adapted, not blindly applied
- **Check for existing scaffolding** — some frameworks have built-in generators (`rails generate`, `nest generate`) that may be better than a custom skill
