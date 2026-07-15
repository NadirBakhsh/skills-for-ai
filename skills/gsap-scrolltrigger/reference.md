# Reference — gsap-scrolltrigger

## Source

- **Repository**: https://github.com/greensock/gsap-skills
- **Skill path**: `skills/gsap-scrolltrigger/SKILL.md`
- **License**: MIT (GreenSock)
- **Docs**: https://gsap.com/docs/v3/Plugins/ScrollTrigger/

## Related skills

Part of the official GSAP skillstack — see [gsap-skillstack/catalog.md](../gsap-skillstack/catalog.md) for the full set and install commands.

| Skill | Use when |
|-------|----------|
| `gsap-core` | Tweens, eases, basic GSAP API |
| `gsap-timeline` | Sequenced multi-step animations |
| `gsap-react` | React/Next.js cleanup with `useGSAP()` |
| `gsap-plugins` | ScrollSmoother, ScrollTo, Flip, etc. |
| `gsap-frameworks` | Vue/Svelte lifecycle |
| `gsap-performance` | Performance optimization |
| `gsap-utils` | `gsap.utils` helpers |

## Install into Cursor

```bash
# From this skills-for-ai repo
cp -r skills/gsap-scrolltrigger ~/.cursor/skills/

# Or full GSAP stack
for skill in gsap-core gsap-timeline gsap-scrolltrigger gsap-plugins gsap-utils gsap-react gsap-performance gsap-frameworks; do
  cp -r "skills/$skill" ~/.cursor/skills/
done

# Or via npx skills
npx skills add greensock/gsap-skills --skill gsap-scrolltrigger -a cursor -y
```

| Scope | Path |
|-------|------|
| Personal (all projects) | `~/.cursor/skills/gsap-scrolltrigger/` |
| Project-only | `.cursor/skills/gsap-scrolltrigger/` |

## Package dependency

```bash
pnpm add gsap
```

Register in your app:

```javascript
import gsap from "gsap";
import { ScrollTrigger } from "gsap/ScrollTrigger";

gsap.registerPlugin(ScrollTrigger);
```

For React/Next.js, also install `@gsap/react` and use the `useGSAP()` hook for cleanup.

## Sync from upstream

```bash
curl -fsSL https://raw.githubusercontent.com/greensock/gsap-skills/main/skills/gsap-scrolltrigger/SKILL.md \
  -o skills/gsap-scrolltrigger/SKILL.md
```

## vs claudedesignskills `gsap-scrolltrigger`

Prefer this **official GreenSock** skill for ScrollTrigger API accuracy. The claudedesignskills version includes generator scripts but is community-maintained — use official for scroll trigger config, pinning, scrub, and `containerAnimation` patterns.
