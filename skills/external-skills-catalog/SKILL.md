---
name: external-skills-catalog
description: >-
  Installs and removes curated third-party agent skills via the Skills CLI
  (npx skills). Use when the user asks to add, remove, list, or update external
  skills from tanstack-skills, vercel-labs, anthropics, or remotion-dev — or
  when managing the recommended external skill set for this project.
---

# External Skills Catalog

Curated third-party skills installed via [Skills CLI](https://skills.sh/) (`npx skills`). Run all commands from the **project root** (`skills-for-ai`).

**Do not auto-install** unless the user explicitly asks. Provide the command from [catalog.md](catalog.md) and let the user run it.

## Quick commands

```bash
# List what's installed
npx skills list

# Search for more skills
npx skills find <query>

# Install all recommended (project + Cursor) — see catalog.md
# Remove all recommended — see catalog.md
```

## Recommended external skills

| Skill | Repo | Use when |
|-------|------|----------|
| `tanstack-devtools` | tanstack-skills/tanstack-skills | TanStack Query/Table/Router devtools setup |
| `find-skills` | vercel-labs/skills | Discover and install other skills |
| `frontend-design` | anthropics/skills | Frontend UI/design implementation |
| `vercel-react-best-practices` | vercel-labs/agent-skills | React/Next.js performance patterns from Vercel |
| `remotion-best-practices` | remotion-dev/skills | Remotion video-in-React projects |

Full **add** and **remove** commands: [catalog.md](catalog.md)

## Install workflow

1. `cd` to project root.
2. Run the **add** command for the skill (from catalog).
3. Verify: `npx skills list`
4. Commit `skills-lock.json` if created (team sync).

## Remove workflow

1. Run the matching **remove** command from catalog.
2. Verify: `npx skills list`
3. Commit updated `skills-lock.json`.

## Rules

- Target **Cursor** with `-a cursor` unless the user names another agent.
- Default to **project scope** (omit `-g`). Use `-g` only when the user wants personal/global install.
- Pass **`-y`** to skip prompts in scripts and automation.
- Never hand-edit `skills-lock.json` — let the CLI manage it.
- Skills from this repo (`skills/*`) are copied manually; external skills use `npx skills add`.

## Related

- Local frontend stack: [Recommended-Stack/Frontend/README.md](../Recommended-Stack/Frontend/README.md)
- Local frontend AI rules: [Recommended-Stack/Frontend/AI-CODING-RULES.md](../Recommended-Stack/Frontend/AI-CODING-RULES.md)
- Install details and bulk scripts: [reference.md](reference.md)
