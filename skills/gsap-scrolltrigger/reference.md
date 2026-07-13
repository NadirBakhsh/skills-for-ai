# Reference — gsap-scrolltrigger

## Source

- **Repository**: https://github.com/greensock/gsap-skills
- **Skill path**: `skills/gsap-scrolltrigger/SKILL.md`
- **License**: MIT (GreenSock)
- **Docs**: https://gsap.com/docs/v3/Plugins/ScrollTrigger/

## Install into Cursor

```bash
# From this skills-for-ai repo
cp -r skills/gsap-scrolltrigger ~/.cursor/skills/

# Or fetch upstream directly
mkdir -p ~/.cursor/skills/gsap-scrolltrigger
curl -fsSL https://raw.githubusercontent.com/greensock/gsap-skills/main/skills/gsap-scrolltrigger/SKILL.md \
  -o ~/.cursor/skills/gsap-scrolltrigger/SKILL.md
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

## Related official GSAP skills

From [greensock/gsap-skills](https://github.com/greensock/gsap-skills) — install alongside as needed:

| Skill | Use when |
|-------|----------|
| `gsap-core` | Tweens, eases, basic GSAP API |
| `gsap-timeline` | Sequenced multi-step animations |
| `gsap-react` | React/Next.js cleanup with `useGSAP()` |
| `gsap-plugins` | ScrollSmoother, ScrollTo, other plugins |
| `gsap-frameworks` | Framework-specific integration |
| `gsap-performance` | Performance optimization |
| `gsap-utils` | Utility patterns |

```bash
REPO=https://raw.githubusercontent.com/greensock/gsap-skills/main/skills
for skill in gsap-core gsap-timeline gsap-react gsap-plugins; do
  mkdir -p ~/.cursor/skills/$skill
  curl -fsSL "$REPO/$skill/SKILL.md" -o ~/.cursor/skills/$skill/SKILL.md
done
```

## Sync from upstream

```bash
curl -fsSL https://raw.githubusercontent.com/greensock/gsap-skills/main/skills/gsap-scrolltrigger/SKILL.md \
  -o skills/gsap-scrolltrigger/SKILL.md
```

## vs claudedesignskills `gsap-scrolltrigger`

Prefer this **official GreenSock** skill for ScrollTrigger API accuracy. The claudedesignskills version includes generator scripts but is community-maintained — use official for scroll trigger config, pinning, scrub, and `containerAnimation` patterns.
