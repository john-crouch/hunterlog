# ABOUTME: Project-local skills index for hunterlog
# ABOUTME: React 18 + MUI v5 + Leaflet skills specific to this project

# Hunterlog Skills

Project-specific skills for the hunterlog codebase.

## Available Skills

| Skill | Coverage |
|-------|----------|
| `mui-v5-conventions/` | React 18 + TypeScript + MUI v5 patterns, styling, accessibility |
| `leaflet-ui-patterns/` | Leaflet map UI, markers, layers, performance |
| `pywebview-bridge/` | pywebview JS↔Python bridge architecture, typed API wrappers |

## By Problem Type

| Problem | Skill |
|---------|-------|
| "MUI styling/sx prop?" | `mui-v5-conventions` |
| "React component patterns?" | `mui-v5-conventions` |
| "Accessibility (a11y)?" | `mui-v5-conventions` → references/accessibility.md |
| "Leaflet map performance?" | `leaflet-ui-patterns` |
| "Map markers/layers?" | `leaflet-ui-patterns` |
| "pywebview bridge API?" | `pywebview-bridge` |
| "JS↔Python communication?" | `pywebview-bridge` |

## Skill Structure

```
.claude/skills/
├── _INDEX.md                    ← You are here
├── mui-v5-conventions/
│   ├── SKILL.md                 ← Main MUI guide
│   └── references/
│       ├── accessibility.md     ← A11y patterns
│       └── component-patterns.md
├── leaflet-ui-patterns/
│   ├── SKILL.md                 ← Map UI patterns
│   └── references/
│       └── EXAMPLES.md          ← Code examples
└── pywebview-bridge/
    └── SKILL.md                 ← JS↔Python bridge patterns
```

## Related User-Wide Skills

These skills from `~/.claude/skills/` complement the project skills:
- `_PATTERNS.md` - Cross-language architectural patterns
- `source-control/` - Git workflow and conventional commits
