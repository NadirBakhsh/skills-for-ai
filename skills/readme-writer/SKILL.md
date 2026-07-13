---
name: readme-writer
description: >-
  Creates or refreshes a project README.md with a complete, consistent
  structure. Use when the user asks to write, generate, or improve a README,
  or when starting a new project that needs documentation.
---

# README Writer

## Quick Start

1. Inspect the project: package manifest (`package.json`, `pyproject.toml`, etc.), entry points, existing scripts, and folder structure.
2. Determine the project type (library, CLI, web app, API service) — this changes which sections matter most.
3. Draft the README using the structure below, omitting sections that don't apply.
4. Verify every command shown (install, run, test) actually matches what's in the repo — never invent commands or package names.

## Structure

```markdown
# Project Name

One or two sentence description of what it does and who it's for.

## Features
- Key capability 1
- Key capability 2

## Installation
[Exact install commands for this repo's package manager]

## Usage
[Minimal working example]

## Configuration
[Env vars / config file options, only if applicable]

## Development
[How to run locally, run tests, build]

## License
[License name, if a LICENSE file exists]
```

## Rules

- Never fabricate badges, license names, or commands — only include what's verifiable from the repo.
- Prefer real, runnable code snippets over abstract descriptions.
- If a section has no content for this project, omit it entirely rather than leaving a placeholder.
- Match the package manager already in use (npm/pnpm/yarn/pip/poetry) — don't mix them.
