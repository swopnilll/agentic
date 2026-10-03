# Claude Code Skills: Worked Example (Surplus Food API)

Example project: backend for a surplus food ordering app, using **NestJS + TypeScript**, **pnpm**, **Prisma** and **PostgreSQL**.

## Step 1: Set up the repo yourself first

Claude Code doesn't pick the framework for you. You decide that, then add the Claude files on top.

```bash
pnpm dlx @nestjs/cli new surplus-food-api --package-manager pnpm
cd surplus-food-api
git init
```

Now you have a normal NestJS + TypeScript project.

## Step 2: Write `CLAUDE.md`

`CLAUDE.md` is for **things that are always true** about the project. Claude reads it every time. Think of it as house rules.

```markdown
# Surplus Food API

Backend for a surplus food ordering app.

## Stack
- NestJS + TypeScript
- PostgreSQL with Prisma
- pnpm only (never npm or yarn)

## Conventions
- One Nest module per feature (orders, listings, vendors, users)
- DTOs use class-validator
- Docs live in /docs
```

**CLAUDE.md = facts and rules. Skills = procedures.**

- "We use pnpm" goes in CLAUDE.md.
- "Here are the steps to add a library" goes in a skill.

## Step 3: Create the skills folder

Each skill is **one folder** with a `SKILL.md` inside:

```
surplus-food-api/
├── CLAUDE.md
├── .claude/
│   └── skills/
│       ├── add-library/
│       │   └── SKILL.md
│       ├── design-schema/
│       │   └── SKILL.md
│       └── generate-orm/
│           └── SKILL.md
├── src/
└── package.json
```

The folder name becomes the command: `/add-library`, `/design-schema`, `/generate-orm`.

## Skill 1: `add-library`

`.claude/skills/add-library/SKILL.md`

```markdown
---
name: add-library
description: Adds a new library to this NestJS project using pnpm. Use when the user wants to install or add a package.
---

1. Check whether NestJS already provides this (e.g. @nestjs/config, @nestjs/jwt) and prefer that.
2. Install with `pnpm add <package>` (use `pnpm add -D` for dev dependencies). Never use npm or yarn.
3. Install the @types package too if needed.
4. Register it in the correct Nest module (don't dump everything in AppModule).
5. If it needs env variables, add them to .env.example.
6. Run `pnpm build` to confirm nothing broke.
7. Tell me what you installed and where you wired it in.
```

## Skill 2: `design-schema`

`.claude/skills/design-schema/SKILL.md`

````markdown
---
name: design-schema
description: Designs or updates the database schema as an ER diagram only. Use when planning tables, entities or relationships. Does not write code.
---

1. Read /docs/schema.md if it exists.
2. Design or update the entities (e.g. User, Vendor, Listing, Order, OrderItem).
3. Output a Mermaid erDiagram and save it to /docs/schema.md.
4. List the relationships in plain words under the diagram.
5. Do NOT create Prisma models or run migrations. Diagram only.
````

## Skill 3: `generate-orm`

`.claude/skills/generate-orm/SKILL.md`

```markdown
---
name: generate-orm
description: Turns the approved schema diagram into Prisma models and creates the migration file without applying it. Use after the schema design is agreed.
---

1. Read /docs/schema.md.
2. Write or update prisma/schema.prisma to match the diagram.
3. Create the migration file only, with a clear name:
   `pnpm prisma migrate dev --create-only --name <clear-migration-name>`
   Always use --create-only. Never apply a migration in this step.
4. Open the generated migration.sql file and inspect it.
5. Summarise the SQL in plain words. Flag anything destructive (DROP TABLE, DROP COLUMN, data loss, type changes).
6. Stop and ask me to confirm.
7. Only after I confirm, apply it with `pnpm prisma migrate dev`, then run `pnpm prisma generate`.
8. Report any mismatch between the diagram and the Prisma models.
```

## How they work together

1. You create the repo and write `CLAUDE.md`.
2. `/add-library`: install the packages you need (Prisma, config, validation).
3. `/design-schema`: Claude draws the ER diagram, and you review and fix it.
4. `/generate-orm`:
   - Claude builds the Prisma models from the diagram.
   - It creates the migration file only (`--create-only`).
   - It inspects the SQL and shows you a summary.
   - It applies the migration only after you confirm.

Skills 2 and 3 are deliberately separate, so you can check the diagram before any code or migration exists. The `--create-only` step is a second checkpoint: you see the real SQL before it touches the database.

Claude only reads the name and description of each skill until it actually uses one, so the long steps cost nothing until then.

## How to use it (will Claude read the skills automatically?)

Mostly yes, but not reliably. For ordered steps like these, call the skill yourself.

### What Claude reads automatically

- The **name and description** of every skill are always loaded, so Claude knows the skills exist.
- The full steps inside `SKILL.md` are loaded only when the skill is actually used.
- Claude decides to use a skill by matching your prompt to its description. If the prompt is vague, it may skip the skill and do the work its own way.

### Why a vague prompt is risky

A prompt like "This is the plan for the user and profile schema. Start working on it" is vague. Claude might use `design-schema`, might jump straight to writing Prisma models, or might do something in between. The skills are built to run in a set order, so don't leave that to chance.

### Better options

1. **Call it explicitly.** This always works:
   ```
   /design-schema  Here is the plan for the user and profile schema: <paste plan>
   ```
   Claude draws the ER diagram into `/docs/schema.md` and stops. Review it, then run:
   ```
   /generate-orm
   ```
2. **Name the skill in plain words**, e.g. "Use the design-schema skill for this plan". This is less strict than a slash command but usually works.
3. **Match the description to how you talk.** If you always say "plan the schema", put those words in the `description` so automatic matching works better.

### Before you try it

- Save the files as `.claude/skills/<skill-name>/SKILL.md` in the repo. Claude Code watches skill folders, so a new skill should appear in the current session. If it doesn't show up when you type `/`, run **reload plugins**.
- `CLAUDE.md` sits at the repo root, and Claude reads it each session. That's why "pnpm only" belongs there.
- If the plan is already a finished diagram, skip `/design-schema` and go straight to `/generate-orm`.