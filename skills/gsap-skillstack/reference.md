# Reference — gsap-skillstack

## Source

- **Repository**: https://github.com/greensock/gsap-skills
- **License**: MIT (GreenSock)
- **Skill index**: `skills/llms.txt` in upstream repo

## Vendored paths in this repo

| Skill | Path |
|-------|------|
| gsap-core | `skills/gsap-core/SKILL.md` |
| gsap-timeline | `skills/gsap-timeline/SKILL.md` |
| gsap-scrolltrigger | `skills/gsap-scrolltrigger/SKILL.md` |
| gsap-plugins | `skills/gsap-plugins/SKILL.md` |
| gsap-utils | `skills/gsap-utils/SKILL.md` |
| gsap-react | `skills/gsap-react/SKILL.md` |
| gsap-performance | `skills/gsap-performance/SKILL.md` |
| gsap-frameworks | `skills/gsap-frameworks/SKILL.md` |

## Cursor install paths

| Scope | Path |
|-------|------|
| Personal (all projects) | `~/.cursor/skills/<skill-name>/` |
| Project-only | `.cursor/skills/<skill-name>/` |

## vs claudedesignskills GSAP skills

| Concern | Prefer |
|---------|--------|
| ScrollTrigger API, pinning, scrub, `containerAnimation` | **Official** `gsap-scrolltrigger` (this repo) |
| Community generator scripts, broader design stack routing | `claude-design-skillstack` → claudedesignskills |

Use **gsap-skillstack** when the task is GSAP-specific. Use **claude-design-skillstack** when the task spans 3D, design, or multi-library animation.

## vs Motion.dev

| Use GSAP when… | Use Motion.dev when… |
|----------------|----------------------|
| ScrollTrigger, pinning, scrub | React-first layout/gesture animations |
| Framework-agnostic or Vue/Svelte | React/Next.js/Svelte with Motion API |
| Complex timeline choreography | Simple component-level motion |
| SVG morphing, FLIP, Draggable plugins | UI micro-interactions, page transitions |

## Licensing note

Since Webflow's acquisition of GSAP, all plugins (SplitText, MorphSVG, etc.) are free for commercial use. Install from the public `gsap` npm package — no `.npmrc`, auth token, or Club GSAP membership.
