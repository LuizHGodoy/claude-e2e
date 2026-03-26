# claude-e2e

A Claude Code skill for E2E testing web applications using the browser integration.

## What it does

When you run `/e2e`, Claude acts as a QA engineer: it opens your web app in a real browser, tests functionality, responsivity across breakpoints, behavior, visual correctness, and accessibility. Every finding is documented with screenshots and video recordings in a `QA/` directory in your project.

## Prerequisites

- Claude Code v2.0.73+
- Google Chrome or Microsoft Edge with the [Claude in Chrome extension](https://chromewebstore.google.com/detail/claude-in-chrome/dhjpnafoafjjldmmckdlgaijnkfmmjfp) v1.0.36+
- Browser integration enabled (run `/chrome` in Claude Code)

## Installation

Add this plugin to your project's `.claude/settings.json`:

```json
{
  "plugins": [
    "claude-e2e"
  ]
}
```

Or install it via npm (once published):

```bash
npm install -g claude-e2e
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
    └── behavior-login-flow.gif      # Interaction recordings
```

The report includes a summary table, pass/fail verdict, detailed issue descriptions with reproduction steps, a responsive breakpoint matrix, and an evidence index.

## License

MIT
