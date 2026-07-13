# External Skills — Add & Remove Commands

Run from **project root** (`skills-for-ai`). Append `-g` for global (user-level) install.

Common flags:
- `-a cursor` — install/remove for Cursor only
- `-y` — skip confirmation prompts

---

## Per-skill commands

### tanstack-devtools

TanStack devtools integration patterns.

```bash
# Add
npx skills add https://github.com/tanstack-skills/tanstack-skills --skill tanstack-devtools -a cursor -y

# Remove
npx skills remove tanstack-devtools -a cursor -y
```

---

### find-skills

Discover and install skills from the open ecosystem.

```bash
# Add
npx skills add https://github.com/vercel-labs/skills --skill find-skills -a cursor -y

# Remove
npx skills remove find-skills -a cursor -y
```

---

### frontend-design

Anthropic frontend design skill — UI implementation guidance.

```bash
# Add
npx skills add https://github.com/anthropics/skills --skill frontend-design -a cursor -y

# Remove
npx skills remove frontend-design -a cursor -y
```

---

### vercel-react-best-practices

Vercel React/Next.js performance and best practices.

```bash
# Add
npx skills add https://github.com/vercel-labs/agent-skills --skill vercel-react-best-practices -a cursor -y

# Remove
npx skills remove vercel-react-best-practices -a cursor -y
```

---

### remotion-best-practices

Remotion video composition best practices.

```bash
# Add
npx skills add https://github.com/remotion-dev/skills --skill remotion-best-practices -a cursor -y

# Remove
npx skills remove remotion-best-practices -a cursor -y
```

---

## Install all (one-shot)

```bash
npx skills add https://github.com/tanstack-skills/tanstack-skills --skill tanstack-devtools -a cursor -y && \
npx skills add https://github.com/vercel-labs/skills --skill find-skills -a cursor -y && \
npx skills add https://github.com/anthropics/skills --skill frontend-design -a cursor -y && \
npx skills add https://github.com/vercel-labs/agent-skills --skill vercel-react-best-practices -a cursor -y && \
npx skills add https://github.com/remotion-dev/skills --skill remotion-best-practices -a cursor -y
```

## Remove all (one-shot)

```bash
npx skills remove tanstack-devtools find-skills frontend-design vercel-react-best-practices remotion-best-practices -a cursor -y
```

## Verify

```bash
npx skills list
npx skills check    # check for upstream updates
```

## Shorthand (owner/repo format)

Same as full URL — either works:

```bash
npx skills add tanstack-skills/tanstack-skills --skill tanstack-devtools -a cursor -y
npx skills add vercel-labs/skills --skill find-skills -a cursor -y
npx skills add anthropics/skills --skill frontend-design -a cursor -y
npx skills add vercel-labs/agent-skills --skill vercel-react-best-practices -a cursor -y
npx skills add remotion-dev/skills --skill remotion-best-practices -a cursor -y
```

## Links

| Skill | Registry |
|-------|----------|
| tanstack-devtools | https://skills.sh/tanstack-skills/tanstack-skills/tanstack-devtools |
| find-skills | https://skills.sh/vercel-labs/skills/find-skills |
| frontend-design | https://skills.sh/anthropics/skills/frontend-design |
| vercel-react-best-practices | https://skills.sh/vercel-labs/agent-skills/vercel-react-best-practices |
| remotion-best-practices | https://skills.sh/remotion-dev/skills/remotion-best-practices |
