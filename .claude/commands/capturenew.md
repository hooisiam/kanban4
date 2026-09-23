---
description: Capture a fresh screenshot of the Kanban board with the Playwright MCP and embed it in the README
argument-hint: [url]   defaults to the local index.html served on 127.0.0.1:8765
allowed-tools: Bash, Read, Edit, Write, mcp__playwright__browser_navigate, mcp__playwright__browser_resize, mcp__playwright__browser_wait_for, mcp__playwright__browser_take_screenshot, mcp__playwright__browser_close
---

# Capture a new README screenshot

Arguments: `$ARGUMENTS`

Capture the board with the **Playwright MCP** tools (`mcp__playwright__*`, configured in `.mcp.json`) and embed the image in `README.md`. Print a short status line as each step finishes. Stop and report if any step fails.

## 1. Preflight

- If the Playwright MCP tools aren't available (for example, the server failed to start because `npx`/Node.js is missing), tell the user how to fix it (`brew install node`, then restart the session) and **stop**. Don't fall back to another browser unless the user asks. Keep any existing `docs/screenshot.png`.

## 2. Serve the page

- If a URL was given as the argument, use it (e.g. the live GitHub Pages site).
- Otherwise run `python3 -m http.server 8765 --bind 127.0.0.1` in the background from the repo root and use `http://127.0.0.1:8765/index.html`.

## 3. Capture

1. `browser_resize` to 1440 × 900.
2. `browser_navigate` to the URL.
3. `browser_wait_for` the text `UOB-ITPM-0001`, so the seeded cards have rendered.
4. `browser_take_screenshot` of the viewport (not full page) as PNG.
5. Copy the saved file to `docs/screenshot.png` (create `docs/` if it's missing), overwriting the old one.
6. `browser_close`, and stop the HTTP server if you started one.

## 4. Check the image

Open `docs/screenshot.png` with Read. Confirm that it shows the header and all four columns (Backlog, In Progress, Blocked, Done) with cards, and no error page. It must be under 5 MB.

## 5. Update the README

Make sure `README.md` has this section right after the **Live demo** line, and don't duplicate it if it's already there:

```markdown
## Screenshot

![UOB IT PMO Kanban board](docs/screenshot.png)
```

Leave the rest of the README as it is.

## 6. Report

Show the screenshot path, its size and the README diff. Don't commit or push. `/publish` does that.
