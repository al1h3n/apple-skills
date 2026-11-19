# Apple Design Skills for AI Agents

[![skills.sh](https://skills.sh/b/al1h3n/apple-skills)](https://skills.sh/al1h3n/apple-skills)

Three agent skills for building Apple-grade interfaces: the craft, the iPhone and iPad specifics, and the Liquid Glass material. Current as of iOS, iPadOS and macOS 27; version 26 is kept as version files, and a template is ready for 28.

Works with [skills.sh](https://skills.sh) and agents that read skills: Claude Code, OpenCode, Cursor, Codex, Windsurf, Antigravity.

## Skills

| Skill | Responsibility | Load it for |
|---|---|---|
| `apple-ui` | Craft shared by every platform, plus macOS | hierarchy, typography, spacing, colour, motion principles, copy, landing pages, AI assistant UI, review checklist, Mac windows / toolbars / menus |
| `ios` | iPhone and iPad only | choosing components, navigation structure, Dynamic Type, gestures, widgets, Lock Screen, Control Center, notifications |
| `liquid-glass` | The material | layer stacks and exact kit values, design rules, per-component glass spec, measured motion, icons, SwiftUI / UIKit / AppKit APIs, shader maths, the Liquid Rust engine and its Svelte package |

They are split by responsibility: each fact lives in one file and the others link to it. `ios` expects `apple-ui` and `liquid-glass` to be installed.

## Where the numbers come from

- **Apple's Figma libraries** (iOS and iPadOS 26 and 27, macOS 26 and 27), every component read node by node.
- **Eight screen recordings** of iOS 26 and 27, measured frame by frame (press, bars, switch, banners, the assistant, Control Center, app open and close).
- **Apple's documentation and sessions** (HIG, WWDC25, WWDC26).

Every number in the skills carries a tag saying which of these it rests on. See `skills/liquid-glass/references/sources.md`.

## Install

```bash
bunx --yes skills add al1h3n/apple-skills --all -g
```

One skill, chosen agents:

```bash
npx skills add al1h3n/apple-skills -s liquid-glass -g -y -a claude-code opencode
```

Agent locations: Claude Code `~/.claude/skills/<skill>`, OpenCode `~/.config/opencode/skills/<skill>`, Cursor `.cursor/skills`, Codex `.codex/skills`, Windsurf `.windsurf/skills`.

## Layout

```
skills.sh.json
skills/
  apple-ui/
    SKILL.md
    references/      focus-and-hierarchy, typography, layout-and-spacing, color-and-materials,
                     motion-principles, copywriting, marketing-pages, ai-assistant-ui, macos,
                     review-checklist, figma-workflow, versions/26, versions/27
  ios/
    SKILL.md
    references/      components, navigation, dynamic-type, gestures-and-motion,
                     widgets-and-system, versions/26, versions/27, versions/_template
  liquid-glass/
    SKILL.md
    assets/          glass.wgsl, pyramid.wgsl
    references/      principles, anatomy, design-rules, components, motion, effects, icons,
                     native-apple, shader-math, webgpu, liquid-rust, bits-ui, sources,
                     versions/26, versions/27, versions/_template
```

## When a new OS ships

The files outside `versions/` describe only the newest OS. To add one: copy `versions/_template.md` in each skill, measure the same situations again, move what changed into the previous version's file, and update the current files. The steps are written in `skills/liquid-glass/references/versions/_template.md`.
