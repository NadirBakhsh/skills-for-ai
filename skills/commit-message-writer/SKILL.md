---
name: commit-message-writer
description: >-
  Generates conventional commit messages by analyzing staged git diffs.
  Use when the user asks for help writing a commit message, wants to commit
  changes, or asks to summarize a diff for a commit.
---

# Commit Message Writer

## Quick Start

1. Run `git diff --staged` (or `git diff` if nothing is staged) to see the actual changes.
2. Identify the primary type of change: `feat`, `fix`, `refactor`, `docs`, `test`, `chore`, `style`, `perf`.
3. Identify the scope: the module, folder, or feature area most affected.
4. Write the message using the format below.

## Format

```
<type>(<scope>): <short summary, imperative mood, no trailing period>

<optional body: what changed and why, wrapped at ~72 chars>
```

- Subject line: max 72 characters.
- Use imperative mood ("add", "fix", "remove" — not "added", "fixes").
- Only add a body when the "why" isn't obvious from the subject alone.
- If changes span multiple unrelated concerns, recommend splitting into separate commits instead of writing one broad message.

## Examples

**Diff**: adds JWT-based login endpoint and middleware.

```
feat(auth): add JWT-based login endpoint

Add token issuance on login and middleware to validate
bearer tokens on protected routes.
```

**Diff**: fixes incorrect timezone conversion in report dates.

```
fix(reports): correct timezone conversion for report dates

Dates were rendered in server-local time instead of UTC.
```

**Diff**: only touches formatting, no logic changes.

```
style(components): reformat with prettier
```
