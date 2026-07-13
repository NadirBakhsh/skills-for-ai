# Reference — Installing claudedesignskills in Cursor

## Repository

- **URL**: https://github.com/freshtechbro/claudedesignskills
- **License**: MIT
- **Skills**: 22 individual + 5 bundles
- **Skill source path**: `.claude/skills/<skill-name>/`
- **Packaged zips**: `.claude/skills/<skill-name>.zip`

## Cursor vs Claude Code Paths

| Platform | Personal skills | Project skills |
|----------|----------------|----------------|
| **Cursor** | `~/.cursor/skills/<name>/` | `.cursor/skills/<name>/` |
| Claude Code | `~/.claude/skills/<name>/` | `.claude/skills/<name>/` |

Content is identical — only the parent directory differs.

## Install Methods

### Method 1: Copy from cloned repo (recommended)

```bash
git clone https://github.com/freshtechbro/claudedesignskills.git
cd claudedesignskills

# Single skill → personal Cursor skills
cp -r .claude/skills/threejs-webgl ~/.cursor/skills/

# Multiple skills
for skill in threejs-webgl gsap-scrolltrigger motion-framer; do
  cp -r ".claude/skills/$skill" ~/.cursor/skills/
done
```

### Method 2: Extract from zip

```bash
unzip claudedesignskills/.claude/skills/threejs-webgl.zip -d ~/.cursor/skills/threejs-webgl/
```

Zip structure (required for claude.ai upload, also works in Cursor):

```
threejs-webgl/
├── SKILL.md          ← at root
├── references/
├── scripts/
└── assets/
```

### Method 3: Sparse checkout (one skill only)

```bash
git clone --depth 1 --filter=blob:none --sparse \
  https://github.com/freshtechbro/claudedesignskills.git
cd claudedesignskills
git sparse-checkout set .claude/skills/threejs-webgl
cp -r .claude/skills/threejs-webgl ~/.cursor/skills/
```

### Method 4: Claude Code plugin marketplace (not Cursor)

For Claude Code CLI only:

```bash
/plugin marketplace add freshtechbro/claudedesignskills
/plugin install threejs-webgl
/plugin install core-3d-animation
```

Cursor users should use Methods 1–3 instead.

## Verify Installation

```bash
# Skill folder must contain SKILL.md
ls ~/.cursor/skills/threejs-webgl/SKILL.md

# Frontmatter must have name + description
head -5 ~/.cursor/skills/threejs-webgl/SKILL.md
```

Expected:

```yaml
---
name: threejs-webgl
description: ...
---
```

## Creating Your Own Skills (upstream tooling)

The upstream repo includes `skill-creator` with validation scripts:

```bash
.claude/skills/skill-creator/scripts/init_skill.py my-skill --path .claude/skills
.claude/skills/skill-creator/scripts/quick_validate.py .claude/skills/my-skill
.claude/skills/skill-creator/scripts/package_skill.py .claude/skills/my-skill
```

For Cursor-native skills, use the same structure but target `.cursor/skills/` (or add to your `skills-for-ai` repo).

## Packaging Anti-Patterns (avoid)

- ❌ `skill-name/skill-name/SKILL.md` (nested subdirectory)
- ❌ Nested `.zip` files inside the archive
- ❌ Missing YAML frontmatter (`name`, `description`)

## Updating Skills

```bash
cd claudedesignskills && git pull
cp -r .claude/skills/threejs-webgl ~/.cursor/skills/   # overwrite
```

## Contributing Back

1. Fork [freshtechbro/claudedesignskills](https://github.com/freshtechbro/claudedesignskills)
2. Add/improve skill using `init_skill.py`
3. Validate with `quick_validate.py`
4. Submit PR

For Cursor-specific meta-skills (like this one), contribute to your [skills-for-ai](https://github.com/NadirBakhsh/skills-for-ai) repo instead.
