# Official GSAP Skills — Catalog

Vendored from [greensock/gsap-skills](https://github.com/greensock/gsap-skills) (MIT). All plugins are free — no Club GSAP membership required.

---

## Skills

| Skill | Description | Triggers |
|-------|-------------|----------|
| `gsap-core` | Core API: `to()`, `from()`, `fromTo()`, easing, stagger, `matchMedia` | JS animation library, tweens, easing, reduced motion |
| `gsap-timeline` | Timelines: position parameter, labels, nesting, playback | sequencing, choreograph, multi-step animation |
| `gsap-scrolltrigger` | ScrollTrigger: pinning, scrub, triggers, refresh, cleanup | scroll animation, parallax, pin section |
| `gsap-plugins` | Flip, Draggable, ScrollSmoother, SplitText, MorphSVG, etc. | plugin, scroll-to, flip, draggable, SVG |
| `gsap-utils` | `gsap.utils`: clamp, mapRange, snap, wrap, interpolate | gsap.utils, clamp, mapRange, random |
| `gsap-react` | React: `useGSAP`, refs, `gsap.context()`, SSR cleanup | React animation, Next.js animation, useGSAP |
| `gsap-performance` | Transforms, will-change, batching, ScrollTrigger perf tips | 60fps, jank, animation performance |
| `gsap-frameworks` | Vue, Svelte, Nuxt, SvelteKit lifecycle + cleanup | Vue, Svelte, framework lifecycle |

---

## Install from this repo

Run from **project root** (`skills-for-ai`):

```bash
# One skill
cp -r skills/gsap-scrolltrigger ~/.cursor/skills/

# Full stack
for skill in gsap-core gsap-timeline gsap-scrolltrigger gsap-plugins gsap-utils gsap-react gsap-performance gsap-frameworks; do
  cp -r "skills/$skill" ~/.cursor/skills/
done
```

Project-only (shared via git):

```bash
mkdir -p .cursor/skills
cp -r skills/gsap-scrolltrigger .cursor/skills/
```

---

## Install via npx skills (upstream)

```bash
# All GSAP skills
npx skills add https://github.com/greensock/gsap-skills -a cursor -y

# One skill
npx skills add https://github.com/greensock/gsap-skills --skill gsap-scrolltrigger -a cursor -y
```

Shorthand:

```bash
npx skills add greensock/gsap-skills -a cursor -y
npx skills add greensock/gsap-skills --skill gsap-core -a cursor -y
```

---

## Remove

```bash
npx skills remove gsap-core gsap-timeline gsap-scrolltrigger gsap-plugins gsap-utils gsap-react gsap-performance gsap-frameworks -a cursor -y
```

---

## Sync from upstream

Refresh vendored copies in this repo:

```bash
REPO=https://raw.githubusercontent.com/greensock/gsap-skills/main/skills
for skill in gsap-core gsap-timeline gsap-scrolltrigger gsap-plugins gsap-utils gsap-react gsap-performance gsap-frameworks; do
  curl -fsSL "$REPO/$skill/SKILL.md" -o "skills/$skill/SKILL.md"
done
```

Or clone and copy:

```bash
git clone --depth 1 https://github.com/greensock/gsap-skills.git /tmp/gsap-skills
for skill in gsap-core gsap-timeline gsap-scrolltrigger gsap-plugins gsap-utils gsap-react gsap-performance gsap-frameworks; do
  cp "/tmp/gsap-skills/skills/$skill/SKILL.md" "skills/$skill/SKILL.md"
done
```

---

## Package dependency

```bash
pnpm add gsap
# React/Next.js also:
pnpm add @gsap/react
```

---

## Links

| Resource | URL |
|----------|-----|
| Upstream repo | https://github.com/greensock/gsap-skills |
| GSAP docs | https://gsap.com/docs/v3/ |
| ScrollTrigger docs | https://gsap.com/docs/v3/Plugins/ScrollTrigger/ |
