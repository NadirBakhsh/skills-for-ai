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
| [`api-endpoint-calls`](skills/api-endpoint-calls/SKILL.md) | Structures typed API endpoint calls — HTTP client, pure API functions, query-key factories, TanStack Query hooks — for React web and React Native/Expo. |
| [`readme-writer`](skills/readme-writer/SKILL.md) | Creates or refreshes a project README with a consistent, complete structure. |
| [`code-review`](skills/code-review/SKILL.md) | Reviews code changes for correctness, security, and maintainability. |
| [`progress-stepper-rsc-suspense`](skills/progress-stepper-rsc-suspense/SKILL.md) | Sequential progress stepper with RSC, Suspense, and `use()` in Next.js 16+. |
| [`claude-design-skillstack`](skills/claude-design-skillstack/SKILL.md) | Routes 3D/animation/design tasks to [claudedesignskills](https://github.com/freshtechbro/claudedesignskills) and installs them in Cursor. |
| [`motion-dev-animations`](skills/motion-dev-animations/SKILL.md) | Motion.dev animations — spring physics, scroll effects, gestures ([upstream](https://github.com/199-biotechnologies/motion-dev-animations-skill)). |
| [`gsap-skillstack`](skills/gsap-skillstack/SKILL.md) | Routes to official GSAP skills — core, timeline, ScrollTrigger, plugins, React, performance, frameworks ([upstream](https://github.com/greensock/gsap-skills)). |
| [`gsap-core`](skills/gsap-core/SKILL.md) | Official GSAP core API — tweens, easing, stagger, `matchMedia`. |
| [`gsap-timeline`](skills/gsap-timeline/SKILL.md) | Official GSAP timelines — sequencing, position parameter, labels. |
| [`gsap-scrolltrigger`](skills/gsap-scrolltrigger/SKILL.md) | Official GSAP ScrollTrigger — scroll-linked animations, pinning, scrub. |
| [`gsap-plugins`](skills/gsap-plugins/SKILL.md) | Official GSAP plugins — Flip, Draggable, ScrollSmoother, SplitText, MorphSVG, etc. |
| [`gsap-utils`](skills/gsap-utils/SKILL.md) | Official GSAP utils — clamp, mapRange, snap, wrap, interpolate. |
| [`gsap-react`](skills/gsap-react/SKILL.md) | Official GSAP React — `useGSAP`, refs, `gsap.context()`, SSR cleanup. |
| [`gsap-performance`](skills/gsap-performance/SKILL.md) | Official GSAP performance — transforms, will-change, batching. |
| [`gsap-frameworks`](skills/gsap-frameworks/SKILL.md) | Official GSAP frameworks — Vue, Svelte, Nuxt, SvelteKit lifecycle. |
| [`external-skills-catalog`](skills/external-skills-catalog/SKILL.md) | Add/remove curated external skills via `npx skills` ([catalog](skills/external-skills-catalog/catalog.md)). |

## Recommended stack

- **Frontend**: [Recommended-Stack/Frontend/README.md](skills/Recommended-Stack/Frontend/README.md) · [AI coding rules](skills/Recommended-Stack/Frontend/AI-CODING-RULES.md)
- **External skills (npx)**: [external-skills-catalog/catalog.md](skills/external-skills-catalog/catalog.md)

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
