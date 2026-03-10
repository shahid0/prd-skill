# PRD Skill for Claude Code

A Claude Code custom command that generates structured, AI-prototyping-ready Product Requirements Documents. Drop it into any project and run `/prd` to start.

---

## What it does

This skill turns a product idea into a complete PRD optimized for two audiences simultaneously:

1. **Your cross-functional team** — PMs, engineers, and designers get the context they need to ask good questions and make confident decisions
2. **AI prototyping tools** — Cursor, v0, Lovable, bolt.new, and Replit get structured sections (component inventory, data models, API specs, file structure) they can use as a build brief without additional clarification

The skill guides you through a conversational discovery process, then generates a 15-section PRD structured around the Jobs to Be Done (JTBD) framework and atomic User Stories.

---

## The approach: JTBD + User Stories

Most PRDs describe what to build. This skill starts with why people need it.

### Jobs to Be Done (JTBD)

JTBD frames each user need as a situational job:

> "When [situation], I want to [motivation], so I can [expected outcome]."

This matters for AI tools because it prevents technically-correct-but-wrong solutions. When Cursor or Lovable knows *why* a feature exists, it makes better architectural choices — not just syntactically valid ones.

### User Stories

User Stories translate JTBD jobs into atomic, implementable features:

> "As a [role], I want to [action], so that [benefit]."

Each story gets a unique ID (US1, US2…) that cross-references:
- The JTBD job it serves
- The acceptance criteria that verify it's done

### How they work together

JTBD operates at the level of human motivation. User Stories operate at the level of product features. Together, they give your team *and* your AI tools a clear line from "why this exists" → "what to build" → "how to verify it's correct."

---

## Why this PRD format works with AI prototyping tools

Each structural section serves a specific purpose for AI tools:

| Section | What AI tools use it for |
|---------|--------------------------|
| **AI Build Summary** | The tl;dr brief — the first thing Cursor or Replit reads to understand the build target |
| **Component Inventory** | v0, Lovable, and bolt.new generate UI at the component level — this gives them the full list |
| **Data Models** | TypeScript interfaces AI tools use to generate type-safe code without guessing schema |
| **API / Integration Surface** | Endpoint specs AI tools use to generate API clients and server routes |
| **State Management Map** | Tells AI tools what lives in local vs. server state vs. URL params — prevents architectural mistakes |
| **Tech Stack Recommendation** | Locks in the stack so AI tools don't mix incompatible libraries |
| **Suggested File Structure** | An ASCII tree AI tools use as a scaffolding target |
| **Acceptance Criteria** | Binary pass/fail checklists AI agents can verify against generated code |

---

## Installation

### Preferred 

```bash
npx skills add https://github.com/shahid0/prd-skill --skill prd-generator-skill
```

### Option 1: Global (available in all projects)

```bash
mkdir -p ~/.claude/commands
cp SKILL.md ~/.claude/commands/prd.md
```

### Option 2: Project-local (this project only)

```bash
mkdir -p .claude/commands
cp SKILL.md .claude/commands/prd.md
```

That's it. Claude Code will detect the file and register `/prd` as a custom command.

---

## Usage

```
/prd
```

Starts the conversational discovery flow. The skill asks about the problem, users, tech stack, and constraints before generating the full PRD.

```
/prd "checkout flow redesign"
```

Pre-populates the PRD title and jumps straight to clarifying questions. Use this when you already know the feature name.

---

## Example output

See [`examples/example-prd.md`](examples/example-prd.md) for a fully filled-out PRD demonstrating every section.

---

## Customization tips

**For simple features:** The skill omits sections that don't apply — just answer "not applicable" when asked, and those sections won't appear in the output.

**For complex multi-team efforts:** Add a "Dependencies" section after Section 3 to map inter-team handoffs, and expand Section 14 (Open Questions & Risks) with specific team owners.

**For design-heavy features:** Expand Section 6 (Proposed Experience) with more detail — annotated screen lists, interaction flows, and specific accessibility requirements.

**For API-first features:** Expand Sections 8 and 9 (Data Models and API Surface) with request/response examples and error codes.

**Adjusting the tech stack defaults:** Edit the prompt in `SKILL.md` to change the default stack recommendation the skill suggests when users have no preference.

---

## How to update the skill

When you want to modify the PRD template:

1. Edit `SKILL.md` in this repo
2. Re-copy it to wherever you installed it: `cp SKILL.md ~/.claude/commands/prd.md`

---

## Compatible tools

Tested and optimized for output that works well with:
- [Cursor](https://cursor.sh)
- [v0 by Vercel](https://v0.dev)
- [Lovable](https://lovable.dev)
- [bolt.new](https://bolt.new)
- [Replit](https://replit.com)
- Any Claude Code instance
