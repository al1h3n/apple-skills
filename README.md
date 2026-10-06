# Apple Design Skills for AI Agents

[![skills.sh](https://skills.sh/b/al1h3n/apple-skills)](https://skills.sh/al1h3n/apple-skills)

A curated collection of production-grade agent skills for Apple Human Interface Guidelines (HIG), iOS interface design, and Apple's Liquid Glass material system.

Compatible with [skills.sh](https://skills.sh) and all modern AI coding agents, including **Claude Code**, **OpenCode**, **Cursor**, **Codex**, **Windsurf**, and **Antigravity**.

---

## Skills Catalog

| Skill Name | Description | Key Focus |
|---|---|---|
| `liquid-glass` | Apple Liquid Glass design system (iOS/iPadOS/macOS 26–27) | SwiftUI `glassEffect` APIs, WGSL/WebGPU shaders, Snell refraction, squircle SDF, rim lighting, analytic springs |
| `ios` | iOS Human Interface Guidelines & mobile app patterns | Dynamic Type ramp, 8 pt rhythm, ≥ 44 pt touch targets, continuous squircles, sheets, navigation, accessibility |
| `apple-ui` | Apple-grade product UI & high-converting marketing pages | Restraint & hierarchy, SF typography licensing, copywriting, dark/light semantic colors, landing pages |

---

## Quick Installation

You can install any skill globally (`-g`) or for your current project using `bunx` or `npx`:

### 1. Install via Bun (`bunx`)

```bash
# Install Liquid Glass skill to Claude Code and OpenCode globally
bunx --yes skills add al1h3n/apple-skills -s liquid-glass -g -y -a claude-code opencode

# Install iOS skill to Claude Code and OpenCode globally
bunx --yes skills add al1h3n/apple-skills -s ios -g -y -a claude-code opencode

# Install Apple UI skill to Claude Code and OpenCode globally
bunx --yes skills add al1h3n/apple-skills -s apple-ui -g -y -a claude-code opencode

# Install all skills in this repository
bunx --yes skills add al1h3n/apple-skills --all -g
```

### 2. Install via npm (`npx`)

```bash
# Install specific skill
npx skills add al1h3n/apple-skills -s liquid-glass -g -y -a claude-code opencode
npx skills add al1h3n/apple-skills -s ios -g -y -a claude-code opencode
npx skills add al1h3n/apple-skills -s apple-ui -g -y -a claude-code opencode
```

---

## Supported AI Agents

The skills CLI installs configuration directly into the respective agent locations:

- **Claude Code**: `~/.claude/skills/<skill-name>`
- **OpenCode**: `~/.config/opencode/skills/<skill-name>` (and universal `.agents/skills`)
- **Cursor**: `.cursor/skills`
- **Codex**: `.codex/skills`
- **Windsurf**: `.windsurf/skills`
- **Antigravity**: Workspace and global skill paths

---

## Repository Structure

```
.
├── skills.sh.json               # skills.sh catalog & grouping definition
├── README.md                    # Installation and documentation
├── skills/                      # Canonical skills directory (skills.sh standard)
│   ├── liquid-glass/            # Apple Liquid Glass material & shader skill
│   │   ├── SKILL.md
│   │   ├── assets/              # WGSL WebGPU shaders
│   │   └── references/          # Math, anatomy, SwiftUI APIs, Liquid Rust
│   ├── ios/                     # iOS HIG mobile pattern skill
│   │   ├── SKILL.md
│   │   └── references/          # Typography ramp, 8pt grid, touch targets
│   └── apple-ui/                # Apple UI & product marketing skill
│       ├── SKILL.md
│       └── references/          # Focus, hierarchy, copywriting, checklists
├── liquid-glass-skill/          # Root package (backward-compatibility)
└── apple-ui-skill/              # Root package (backward-compatibility)
```
