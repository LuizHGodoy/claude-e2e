# claude-e2e

A Claude Code plugin for E2E testing web applications using [agent-browser](https://github.com/vercel-labs/agent-browser).

## What it does

When you run `/e2e`, Claude acts as a QA engineer: it opens your web app in a real browser, tests functionality, responsivity across breakpoints, behavior, visual correctness, and accessibility. Every finding is documented with screenshots and video recordings in a `QA/` directory in your project.

## Prerequisites

- Claude Code v2.0.73+
- [agent-browser](https://github.com/vercel-labs/agent-browser) installed (`npm i -g agent-browser && agent-browser install`)

## Installation

### Via Claude Code CLI

```bash
claude /install-plugin https://github.com/LuizHGodoy/claude-e2e.git
```

### Manual installation

Add this to your `~/.claude/settings.json` (or `~/.claude-{profile}/settings.json`):

```json
{
  "extraKnownMarketplaces": {
    "claude-e2e": {
      "source": {
        "source": "github",
        "repo": "LuizHGodoy/claude-e2e"
      }
    }
  },
  "enabledPlugins": {
    "claude-e2e@claude-e2e": true
  }
}
```

Then restart Claude Code. The plugin will be cloned and cached automatically. Updates are pulled on restart.

### As a project skill (no plugin system)

Copy the skill directly into your project:

```bash
mkdir -p .claude/skills
cp -r skills/e2e .claude/skills/e2e
```

Or symlink for personal use across all projects:

```bash
ln -sf /path/to/claude-e2e/skills/e2e ~/.claude/skills/e2e
```

## Usage

```
/e2e http://localhost:3000
/e2e http://localhost:3000 login flow
/e2e verify the signup fix
```

## What gets tested

| Category | What's checked |
|----------|---------------|
| **Functional** | Core user flows, forms, auth, CRUD, edge cases, error handling |
| **Responsivity** | 8 breakpoints from 320px to 1920px — layout, text, touch targets, navigation |
| **Visual/UI** | Spacing, alignment, color contrast, loading states, hover/focus states |
| **Behavior** | JS interactions, animations, form validation, console errors, navigation |
| **Accessibility** | Semantic HTML, keyboard nav, ARIA labels, alt text, accessibility tree |

## Output

Tests produce a `QA/` directory in your project:

```
QA/
├── reports/
│   └── 2026-03-26-full-suite.md    # Detailed test report
├── screenshots/
│   └── responsive-header-375px.png  # Evidence screenshots
└── videos/
    └── behavior-login-flow.webm     # Interaction recordings
```

The report includes a summary table, pass/fail verdict, detailed issue descriptions with reproduction steps, a responsive breakpoint matrix, and an evidence index.

## License

MIT
