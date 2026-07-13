---
name: claude-design-skillstack
description: >-
  Routes design and 3D/animation tasks to the correct skill from the
  freshtechbro/claudedesignskills collection (22 skills: Three.js, GSAP, R3F,
  Framer Motion, scroll effects, Lottie, etc.). Use when building modern web
  design, 3D scenes, scroll animations, micro-interactions, scrollytelling,
  WebGL, or interactive landing pages — or when the user mentions
  claudedesignskills, design skillstack, or needs to install design skills
  into Cursor.
---

# Claude Design Skillstack (Cursor)

Meta-skill for the [freshtechbro/claudedesignskills](https://github.com/freshtechbro/claudedesignskills) collection — 22 production skills for 3D/WebGL, animation, and modern web design (MIT).

**Upstream**: [github.com/freshtechbro/claudedesignskills](https://github.com/freshtechbro/claudedesignskills)

## Quick Start

1. **Identify the task** → use the [Skill Picker](#skill-picker) below.
2. **Install the skill** into Cursor (see [Install into Cursor](#install-into-cursor)).
3. **Read the installed skill's `SKILL.md`** before implementing — each skill has patterns, scripts, and references.
4. **Combine skills** when needed — see [Common Stacks](#common-stacks).

## Skill Picker

| User wants… | Install this skill |
|-------------|-------------------|
| Three.js scene, WebGL, PBR materials | `threejs-webgl` |
| GSAP timelines, ScrollTrigger, scroll scrub | `gsap-scrolltrigger` |
| React Three Fiber, drei, 3D in React | `react-three-fiber` |
| Framer Motion, layout animations, gestures | `motion-framer` |
| Babylon.js game engine, physics | `babylonjs-engine` |
| Lightweight hero backgrounds (waves, birds, fog) | `lightweight-3d-effects` |
| WebXR / VR in browser | `aframe-webxr` |
| PlayCanvas editor pipeline | `playcanvas-engine` |
| 2D WebGL sprites, particles | `pixijs-2d` |
| Smooth scroll + parallax (Locomotive) | `locomotive-scroll` |
| SPA page transitions | `barba-js` |
| Spring physics in React | `react-spring-physics` |
| Magic UI / shadcn-style animated components | `animated-component-libraries` |
| AOS-style scroll reveals | `scroll-reveal-libraries` |
| Anime.js timelines | `animejs` |
| Lottie JSON animations | `lottie-animations` |
| Blender → glTF for web | `blender-web-pipeline` |
| Spline 3D embeds | `spline-interactive` |
| Rive state machines | `rive-interactive` |
| Substance 3D textures | `substance-3d-texturing` |
| **Multi-library 3D architecture** | `web3d-integration-patterns` |
| **Modern design trends, a11y, performance** | `modern-web-design` |

Full catalog with bundles: [catalog.md](catalog.md)

## Install into Cursor

Skills from claudedesignskills live in `.claude/skills/<name>/`. Copy into Cursor paths:

| Scope | Target path |
|-------|-------------|
| Personal (all projects) | `~/.cursor/skills/<skill-name>/` |
| Project-only | `.cursor/skills/<skill-name>/` |

```bash
# Clone once
git clone https://github.com/freshtechbro/claudedesignskills.git /tmp/claudedesignskills

# Install one skill (personal)
cp -r /tmp/claudedesignskills/.claude/skills/threejs-webgl ~/.cursor/skills/

# Or from zip
unzip /tmp/claudedesignskills/.claude/skills/threejs-webgl.zip -d ~/.cursor/skills/threejs-webgl/
```

**Windows (Git Bash):** same commands; `~` resolves to your home directory.

After install, invoke by name in chat: *"use threejs-webgl"* or describe the task and let the agent match the description.

## Common Stacks

| Project type | Recommended bundle / skills |
|--------------|----------------------------|
| 3D product landing | `threejs-webgl` + `gsap-scrolltrigger` + `modern-web-design` |
| React 3D app | `react-three-fiber` + `motion-framer` + `web3d-integration-patterns` |
| Animated marketing site | `gsap-scrolltrigger` + `scroll-reveal-libraries` + `modern-web-design` |
| Lightweight hero only | `lightweight-3d-effects` + `motion-framer` |
| Full design agency stack | Install bundle `core-3d-animation` (Claude Code) or copy all 5 skills manually |

Bundle contents and Claude Code plugin commands: [catalog.md](catalog.md#bundles)

## Design Principles (summary)

When no specific library skill is installed yet, apply these defaults from `modern-web-design`:

1. **Performance-first** — LCP < 2.5s, CLS < 0.1; defer non-critical animation; lazy-load 3D/assets.
2. **Bold minimalism** — fluid `clamp()` typography, 3–5 color tokens, ample whitespace.
3. **Micro-interactions** — hover/tap feedback (200–300ms); skeleton loaders over spinners.
4. **Scrollytelling** — ScrollTrigger or Motion for reveal-on-scroll; pin + scrub for narrative sections.
5. **Accessibility** — WCAG AA minimum; `prefers-reduced-motion` fallbacks; keyboard-navigable interactions.

Full trend guide lives in upstream `modern-web-design/SKILL.md` — install that skill for detailed patterns.

## Integration Patterns (summary)

From `web3d-integration-patterns` — pick one architecture:

| Pattern | Stack | Best for |
|---------|-------|----------|
| Layered separation | Three.js + GSAP + React/Motion UI | Scroll-driven 3D with HTML overlay |
| R3F declarative | React Three Fiber + Motion | State-driven 3D in React apps |
| R3F + GSAP | R3F + GSAP ScrollTrigger | Complex scrubbed 3D sequences in React |
| R3F + Spring | R3F + React Spring | Physics-based 3D interactions |

Install `web3d-integration-patterns` for full code templates and performance rules.

## Workflow

```
Task Progress:
- [ ] Classify task (3D / animation / scroll / design system / integration)
- [ ] Pick skill from Skill Picker (install if missing)
- [ ] Read installed SKILL.md + run any generator scripts it provides
- [ ] Implement with performance + a11y checks
- [ ] Add second skill only if integration crosses libraries
```

## Rules

1. **Install before deep implementation** — upstream skills contain generator scripts (`setup_scene.py`, `component_generator.py`, etc.) that save time.
2. **One primary skill per concern** — don't mix Three.js raw and R3F in the same component without `web3d-integration-patterns`.
3. **Respect `prefers-reduced-motion`** — disable scroll-scrub and autoplay when set.
4. **Attribute upstream** — skills are MIT-licensed from [freshtechbro/claudedesignskills](https://github.com/freshtechbro/claudedesignskills).
5. **Cursor path is `.cursor/skills/`** — not `.claude/skills/` (that's Claude Code).

## Additional Resources

- Full skill catalog and bundles: [catalog.md](catalog.md)
- Install options, zip structure, validation: [reference.md](reference.md)
