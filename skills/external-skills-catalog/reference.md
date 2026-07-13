# Reference — Skills CLI

Docs: https://vercel-labs-skills.mintlify.app/quickstart · Registry: https://skills.sh/

## Where skills land

The CLI installs to agent-specific paths. For Cursor (project scope):

```
.cursor/skills/<skill-name>/
```

Global install (`-g`) uses user-level paths (e.g. `~/.cursor/skills/`).

## Lock file

Project installs may create `skills-lock.json`. Commit it for team sync. Do not edit manually.

```bash
# Restore from lock file (team / CI)
npx skills experimental_install
```

## Useful commands

| Command | Purpose |
|---------|---------|
| `npx skills list` | List installed skills |
| `npx skills find <query>` | Search registry |
| `npx skills add <repo> --skill <name>` | Install one skill |
| `npx skills remove <name>` | Remove one skill |
| `npx skills check` | Check for updates |
| `npx skills update` | Update installed skills |
| `npx skills remove --all -y` | Remove everything (careful) |

## This repo vs external skills

| Source | Install method |
|--------|----------------|
| `skills-for-ai/skills/*` (local) | `cp -r skills/<name> ~/.cursor/skills/` or `.cursor/skills/` |
| External (this catalog) | `npx skills add ...` from [catalog.md](catalog.md) |

Local skills in this repo are **not** managed by `npx skills remove` — only CLI-installed external skills are.

## Already installed locally (manual copy)

These were installed earlier via `git clone` + `cp`, not `npx skills`:

- claudedesignskills (23 skills in `~/.cursor/skills/`)
- motion-dev-animations
- gsap-scrolltrigger (official)

To remove manual copies:

```bash
rm -rf ~/.cursor/skills/<skill-folder-name>
```
