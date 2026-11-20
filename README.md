# Tweakpane Agent Skill

[![skills.sh](https://skills.sh/b/AddeyX/tweakpane-skill)](https://skills.sh/AddeyX/tweakpane-skill)

An [Agent Skill](https://skills.sh) that teaches AI coding agents the real **Tweakpane v4** API — verified directly against [`cocopon/tweakpane`](https://github.com/cocopon/tweakpane) source so agents stop hallucinating v3 methods, wrong error messages, or missing `refresh()` echo guards.

> **Based on:** `tweakpane@4.0.5` & `@tweakpane/core@2.0.5`

## Install

```bash
npx skills add AddeyX/tweakpane-skill
```

Works with Cursor, Claude Code, Codex, and other Agent Skills-compatible tools.

## What's inside

| File | Covers |
| --- | --- |
| [`skills/tweakpane/SKILL.md`](skills/tweakpane/SKILL.md) | Setup, ESM/TS notes, pane lifecycle, `dispose()`, `refresh()` echo guard, events, v3 → v4 cheatsheet |
| [`skills/tweakpane/bindings.md`](skills/tweakpane/bindings.md) | Plugin check order, widget matrix, numbers, colors, 2D/3D/4D points, live binding APIs, readonly monitors |
| [`skills/tweakpane/blades.md`](skills/tweakpane/blades.md) | Folders, buttons, tabs, unbound blades (`slider`, `list`, `text`, `separator`), state import/export, `--tp-*` CSS variables |
| [`skills/tweakpane/plugins.md`](skills/tweakpane/plugins.md) | `registerPlugin`, official plugins (`essentials`, `camerakit`), bundle contract, `accept()` signatures |

## License

MIT
