# Claude Design Skillstack — Full Catalog

Source: [freshtechbro/claudedesignskills](https://github.com/freshtechbro/claudedesignskills) (MIT)

## Core 3D & Animation (5)

| Skill | Triggers / use cases |
|-------|---------------------|
| `threejs-webgl` | Three.js scenes, WebGL renderer, PBR materials, lights, cameras, glTF loading |
| `gsap-scrolltrigger` | GSAP timelines, ScrollTrigger pin/scrub, scroll-linked animations |
| `react-three-fiber` | R3F components, drei helpers, 3D in React/Next.js |
| `motion-framer` | Framer Motion layout, gestures, page transitions, micro-interactions |
| `babylonjs-engine` | Babylon.js scenes, physics, game-style 3D |

## Extended 3D & Scroll (6)

| Skill | Triggers / use cases |
|-------|---------------------|
| `aframe-webxr` | WebXR, VR scenes, A-Frame entities |
| `lightweight-3d-effects` | Vanta.js backgrounds, low-poly hero effects |
| `playcanvas-engine` | PlayCanvas editor exports, game engine web deploy |
| `pixijs-2d` | 2D WebGL sprites, particles, canvas games |
| `locomotive-scroll` | Smooth scroll, parallax, scroll containers |
| `barba-js` | SPA page transitions between routes |

## Animation & Components (5)

| Skill | Triggers / use cases |
|-------|---------------------|
| `react-spring-physics` | Spring physics, draggable UI, natural motion |
| `animated-component-libraries` | Magic UI, Aceternity, animated shadcn patterns |
| `scroll-reveal-libraries` | AOS, ScrollReveal, fade/slide on scroll |
| `animejs` | Anime.js timelines, SVG morphing, stagger |
| `lottie-animations` | Lottie JSON, After Effects exports on web |

## 3D Authoring & Motion (4)

| Skill | Triggers / use cases |
|-------|---------------------|
| `blender-web-pipeline` | Blender → glTF/GLB, baking, web export |
| `spline-interactive` | Spline 3D scene embeds and interactions |
| `rive-interactive` | Rive state machines, interactive vectors |
| `substance-3d-texturing` | Substance materials for web 3D |

## Meta-Skills (2)

| Skill | Triggers / use cases |
|-------|---------------------|
| `web3d-integration-patterns` | Multi-library 3D architecture, Three+GSAP+R3F combos |
| `modern-web-design` | 2024–2025 design trends, a11y, performance, design systems |

## Bundles

For Claude Code CLI, install bundles via `/plugin install <bundle>`. For Cursor, copy each skill folder individually.

| Bundle | Skills included |
|--------|----------------|
| `core-3d-animation` | threejs-webgl, gsap-scrolltrigger, react-three-fiber, motion-framer, babylonjs-engine |
| `extended-3d-scroll` | aframe-webxr, lightweight-3d-effects, playcanvas-engine, pixijs-2d, locomotive-scroll, barba-js |
| `animation-components` | react-spring-physics, animated-component-libraries, scroll-reveal-libraries, animejs, lottie-animations |
| `authoring-motion` | blender-web-pipeline, spline-interactive, rive-interactive, substance-3d-texturing |
| `meta-skills` | web3d-integration-patterns, modern-web-design |

## Skill Relationships

**Foundations** (install first):
- `threejs-webgl` → base for R3F, A-Frame, Vanta
- `gsap-scrolltrigger` → pairs with most scroll/3D work
- `motion-framer` → base for component animation libs

**Alternatives** (pick one per concern):
- 3D engine: `threejs-webgl` vs `babylonjs-engine` vs `playcanvas-engine`
- Animation: `gsap-scrolltrigger` vs `motion-framer` vs `react-spring-physics`
- Scroll: `locomotive-scroll` vs `scroll-reveal-libraries` (AOS)

**Proven integrations**:
- Three.js + GSAP ScrollTrigger
- R3F + Framer Motion
- Vanta + GSAP

## Generator Scripts (per skill)

Upstream skills include Python generators. Run from the installed skill directory:

| Skill | Script | Purpose |
|-------|--------|---------|
| threejs-webgl | `scripts/setup_scene.py` | Three.js boilerplate |
| react-three-fiber | `scripts/component_generator.py` | 12 R3F component types |
| motion-framer | `scripts/animation_generator.py` | 11 animation types |
| babylonjs-engine | `scripts/scene_generator.py` | 8 scene types |
| gsap-scrolltrigger | `scripts/generate_animation.py` | Scroll animation boilerplate |

## Example Prompts → Skills

| Prompt | Skill |
|--------|-------|
| "Create a Three.js scene with PBR materials" | `threejs-webgl` |
| "Add GSAP scroll animations to the hero" | `gsap-scrolltrigger` |
| "Build an R3F component with physics" | `react-three-fiber` |
| "Design a modern landing page with micro-interactions" | `modern-web-design` + `motion-framer` |
| "Combine Three.js scroll scene with React UI" | `web3d-integration-patterns` |
