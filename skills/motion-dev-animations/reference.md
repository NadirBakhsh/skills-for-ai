# Reference — motion-dev-animations

## Source

- **Repository**: https://github.com/199-biotechnologies/motion-dev-animations-skill
- **License**: MIT (199 Biotechnologies)
- **Version in this copy**: 3.0.0

## Install into Cursor

```bash
# From this skills-for-ai repo
cp -r skills/motion-dev-animations ~/.cursor/skills/

# Or clone upstream directly
git clone https://github.com/199-biotechnologies/motion-dev-animations-skill.git ~/.cursor/skills/motion-dev-animations
```

| Scope | Path |
|-------|------|
| Personal (all projects) | `~/.cursor/skills/motion-dev-animations/` |
| Project-only | `.cursor/skills/motion-dev-animations/` |

## Package dependency

```bash
pnpm add motion          # React / Next.js / Svelte / Astro
pnpm add motion-v        # Vue only
```

## Skill contents

| Path | Purpose |
|------|---------|
| `SKILL.md` | Core workflow, decision tree, patterns |
| `examples/` | hero-fade-up, scroll-reveal, card-hover, parallax, magnetic-button |
| `reference/` | Full API + spring physics presets |
| `templates/` | Next.js page + component library starters |
| `schema/` | `motion-config.schema.json` |
| `scripts/` | `validate_motion_config.py` |

## Sync from upstream

```bash
git clone --depth 1 https://github.com/199-biotechnologies/motion-dev-animations-skill.git /tmp/motion-dev-animations-skill
cp -r /tmp/motion-dev-animations-skill/{SKILL.md,examples,reference,templates,schema,scripts,LICENSE} skills/motion-dev-animations/
```

## Related skills

- `motion-framer` (claudedesignskills) — broader Framer Motion patterns
- `gsap-scrolltrigger` (claudedesignskills) — complex scroll-scrub / SVG animation

Use **motion-dev-animations** for Motion.dev-specific workflow with templates and validation.
