# skills-for-ai

A personal library of [Cursor Agent Skills](https://cursor.com) — reusable, version-controlled instructions that teach AI coding agents how to perform specific tasks consistently.

## What's a Skill?

A Skill is a folder containing a `SKILL.md` file (plus optional reference docs and scripts) that gives an agent step-by-step domain knowledge for a task — e.g. writing commit messages in a specific style, reviewing code against team standards, or generating a README.

Each skill lives in its own directory:

```
skills/
└── skill-name/
    ├── SKILL.md        # required — instructions + YAML frontmatter
    ├── reference.md    # optional — detailed docs, loaded on demand
    └── scripts/         # optional — helper scripts the agent can run
```

## Skills in This Repo

| Skill | Description |
|---|---|
| [`commit-message-writer`](skills/commit-message-writer/SKILL.md) | Generates conventional commit messages from staged git diffs. |
| [`readme-writer`](skills/readme-writer/SKILL.md) | Creates or refreshes a project README with a consistent, complete structure. |
| [`code-review`](skills/code-review/SKILL.md) | Reviews code changes for correctness, security, and maintainability. |

## Installing a Skill

Copy the skill folder you want into one of these locations:

- **Personal** (all your projects): `~/.cursor/skills/<skill-name>/`
- **Project-only** (shared via git with your team): `.cursor/skills/<skill-name>/`

```bash
# Example: install commit-message-writer as a personal skill
cp -r skills/commit-message-writer ~/.cursor/skills/
```

The agent will pick it up automatically — just reference it by name (e.g. "use the commit-message-writer skill") or let the agent auto-invoke it when it's relevant.

## Adding a New Skill

1. Create `skills/<your-skill-name>/SKILL.md` with YAML frontmatter (`name`, `description`).
2. Keep it under 500 lines; push detailed reference material into separate files.
3. Add a row to the table above.

See any existing skill in this repo for a template to follow.

## License

MIT
