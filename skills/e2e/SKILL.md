---
name: e2e
description: Run E2E tests on a web application using agent-browser. Tests functionality, responsivity, behavior, accessibility, and visual correctness. Creates a QA directory with test reports, screenshots, and video evidence. TRIGGER when the user asks to test a web app, verify a fix, check responsivity, run QA, or validate a feature end-to-end.
argument-hint: [url-or-description]
allowed-tools: Bash(agent-browser *), Bash(agent-browser:*), Bash(npx agent-browser:*), Bash(mkdir *), Read, Write, Glob, Grep, Edit
effort: high
---

# E2E Testing Skill

You are an expert QA engineer. Your job is to thoroughly test a web application using `agent-browser`, document every finding, and save all evidence (screenshots and recordings).

## Input

`$ARGUMENTS` — the URL to test, a description of what to test, or both. Examples:
- `/e2e http://localhost:3000` — full test suite on the app
- `/e2e http://localhost:3000 login flow` — test a specific flow
- `/e2e verify the signup fix` — test a recent fix (infer URL from project context)

If no URL is provided, look for common dev server URLs in the project (package.json scripts, docker-compose, .env files) or ask the user.

## Setup

1. Create the QA output directory structure if it doesn't exist:

```bash
mkdir -p QA/reports QA/screenshots QA/videos
```

2. Verify `agent-browser` is available:

```bash
agent-browser open about:blank && agent-browser close
```

If it fails, tell the user to install it: `npm i -g agent-browser && agent-browser install`

3. Verify the target URL is reachable. If the dev server isn't running, tell the user and suggest the command to start it (infer from package.json, Makefile, etc).

## Browser Workflow

Follow the agent-browser pattern for all interactions:

```bash
# 1. Navigate
agent-browser open <url> && agent-browser wait --load networkidle

# 2. Snapshot to get element refs
agent-browser snapshot -i

# 3. Interact using refs
agent-browser click @e1
agent-browser fill @e2 "text"

# 4. Re-snapshot after any navigation or DOM change
agent-browser snapshot -i
```

**Important**: Refs (`@e1`, `@e2`) are invalidated after page changes. Always re-snapshot after clicks that navigate, form submissions, or dynamic content loading.

## Test Strategy

Plan your tests before executing. Cover these categories as applicable:

### 1. Functional Testing
- Core user flows (navigation, forms, auth, CRUD operations)
- Edge cases (empty states, long text, special characters)
- Error handling (invalid inputs, network errors, 404 pages)
- The specific fix/feature mentioned by the user (highest priority)

**How to test:**
```bash
agent-browser open <url> && agent-browser wait --load networkidle
agent-browser snapshot -i
# interact with forms, buttons, links using refs
agent-browser fill @e1 "test input"
agent-browser click @e2
agent-browser wait --load networkidle
agent-browser snapshot -i  # verify result
```

### 2. Responsivity Testing
Test at these viewport sizes:
- **Desktop**: 1920x1080, 1440x900, 1280x720
- **Tablet**: 1024x768, 768x1024
- **Mobile**: 430x932, 375x812, 320x568

**How to test:**
```bash
# For each breakpoint:
agent-browser set viewport 375 812
agent-browser screenshot QA/screenshots/responsive-{page}-375px.png
agent-browser snapshot -i  # check interactive elements still accessible

agent-browser set viewport 1920 1080
agent-browser screenshot QA/screenshots/responsive-{page}-1920px.png
```

For each breakpoint, check:
- Layout doesn't break or overflow
- Text remains readable
- Interactive elements are tappable (minimum 44x44px touch targets)
- Navigation adapts (hamburger menu, collapsible sections)
- Images scale properly
- No horizontal scroll unless intentional

### 3. Visual & UI Testing
- Consistent spacing, alignment, and typography
- Color contrast (WCAG AA minimum)
- Loading states and transitions
- Dark/light mode if applicable
- Hover/focus/active states on interactive elements

**How to test:**
```bash
# Use annotated screenshots for visual inspection
agent-browser screenshot --annotate QA/screenshots/visual-{page}-annotated.png

# Test dark mode if applicable
agent-browser set media dark
agent-browser screenshot QA/screenshots/visual-{page}-dark.png
agent-browser set media light
```

### 4. Behavior Testing
- JavaScript interactions work correctly
- Animations and transitions are smooth
- Forms validate on submit and on blur
- Console errors — check for JS errors/warnings
- Links navigate to correct destinations
- Modals, dropdowns, and tooltips behave correctly

**How to test:**
```bash
# Record interactive flows
agent-browser record start QA/videos/behavior-{flow-name}.webm
agent-browser snapshot -i
agent-browser click @e1  # trigger interaction
agent-browser wait 1000   # let animation complete
agent-browser snapshot -i  # verify result
agent-browser record stop

# Check console for errors at every page
agent-browser eval 'JSON.stringify(window.__console_errors || "check devtools")'
```

### 5. Accessibility Quick Check
- Semantic HTML structure
- Keyboard navigation (tab order, focus trapping in modals)
- ARIA labels on interactive elements
- Alt text on images

**How to test:**
```bash
# Get full accessibility tree
agent-browser snapshot -i  # the snapshot IS the accessibility tree

# Check images without alt text
agent-browser eval --stdin <<'EVALEOF'
JSON.stringify(
  Array.from(document.querySelectorAll("img"))
    .filter(i => !i.alt)
    .map(i => ({ src: i.src.split("/").pop(), width: i.width }))
)
EVALEOF

# Test keyboard navigation
agent-browser press Tab
agent-browser snapshot -i  # check focus moved correctly
agent-browser press Tab
agent-browser snapshot -i
```

## Evidence Collection

For EVERY test you run:

1. **Screenshot**: Take a screenshot at each meaningful step:
   ```bash
   agent-browser screenshot QA/screenshots/{category}-{test-name}-{step}-{viewport}.png
   ```
   Examples:
   - `agent-browser screenshot QA/screenshots/responsive-homepage-header-375px.png`
   - `agent-browser screenshot QA/screenshots/functional-login-error-message-1440px.png`
   - `agent-browser screenshot --full QA/screenshots/functional-full-page-1920px.png`

2. **Recording**: For interactive flows, record before and stop after:
   ```bash
   agent-browser record start QA/videos/{category}-{test-name}-{viewport}.webm
   # ... perform the interaction flow ...
   agent-browser record stop
   ```

3. **Console output**: Check the browser console regularly:
   ```bash
   agent-browser eval 'JSON.stringify(performance.getEntriesByType("resource").filter(r => r.transferSize === 0).map(r => r.name))'
   ```

4. **Visual diffs**: When comparing states before/after a change:
   ```bash
   agent-browser screenshot QA/screenshots/before.png
   # ... make changes ...
   agent-browser diff screenshot --baseline QA/screenshots/before.png
   ```

## Report Generation

After all tests complete, generate a comprehensive report at `QA/reports/{date}-{scope}.md`:

```markdown
# QA Test Report

- **Date**: {YYYY-MM-DD HH:MM}
- **URL**: {tested URL}
- **Scope**: {what was tested — full suite, specific feature, fix verification}
- **Tester**: Claude Code E2E Skill

## Summary

| Category       | Passed | Failed | Warnings |
|----------------|--------|--------|----------|
| Functional     | X      | X      | X        |
| Responsivity   | X      | X      | X        |
| Visual/UI      | X      | X      | X        |
| Behavior       | X      | X      | X        |
| Accessibility  | X      | X      | X        |
| **Total**      | **X**  | **X**  | **X**    |

## Verdict: PASS / FAIL / PASS WITH WARNINGS

{1-2 sentence overall assessment}

## Critical Issues (must fix)

### {Issue title}
- **Category**: {category}
- **Severity**: Critical / High
- **Description**: {what's wrong}
- **Steps to reproduce**: {numbered steps}
- **Expected**: {what should happen}
- **Actual**: {what actually happens}
- **Evidence**: ![screenshot](../screenshots/{filename})
- **Viewport**: {if responsive issue, which breakpoints}

## Warnings (should fix)

### {Issue title}
- Same structure as above but lower severity

## Passed Tests

{Grouped by category — brief list of what passed with evidence links}

## Responsive Breakpoint Matrix

| Page/Component | 1920 | 1440 | 1280 | 1024 | 768 | 430 | 375 | 320 |
|----------------|------|------|------|------|-----|-----|-----|-----|
| Header/Nav     | OK   | OK   | OK   | OK   | OK  | OK  | OK  | OK  |
| {component}    | ...  | ...  | ...  | ...  | ... | ... | ... | ... |

## Console Errors

{List any JS errors or warnings found in the browser console}

## Evidence Index

| File | Description |
|------|-------------|
| [filename](../screenshots/filename) | {description} |
| [filename](../videos/filename) | {description} |
```

## Execution Guidelines

- **Be methodical**: Test one thing at a time, take evidence, move to the next.
- **Prioritize**: If the user asked about a specific fix or feature, test that FIRST and most thoroughly. Then expand to related areas.
- **Don't assume**: Actually navigate, click, and verify. Don't skip tests because "it probably works."
- **Be specific**: In bug reports, provide exact steps, exact text, exact pixel sizes when relevant.
- **Console matters**: Always check the browser console for errors at every page.
- **Compare viewports**: When testing responsivity, take screenshots at each breakpoint for the same page/component so they can be compared side-by-side.
- **Report honestly**: If something looks slightly off but functional, report it as a warning, not a pass.
- **Always re-snapshot**: After any click, navigation, or DOM change, run `agent-browser snapshot -i` to get fresh refs.
- **Clean up**: Run `agent-browser close` when all tests are done.

## After Testing

1. Close the browser session: `agent-browser close`
2. Present a brief summary to the user with the verdict and critical issues count.
3. Point them to the full report path.
4. If critical issues were found, suggest specific fixes based on what you observed.
