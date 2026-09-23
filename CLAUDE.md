# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A single-file IT Project Management Kanban board, labelled "UOB IT PMO". It is an internal demo/training tool. There is no build, lint or test tooling. To run it, open the HTML file directly in a browser (`file://`); it needs no server.

There are two independent versions, each a complete single file (markup, one `<style>` block, one `<script>` block):

| File | Design | Live URL |
|---|---|---|
| `index.html` | Original design (header summary pills) and a CSP | https://hooisiam.github.io/kanban4/ |
| `v2/index.html` | Redesign with the stacked progress chart, also with a CSP | https://hooisiam.github.io/kanban4/v2/ |

Keep them separate: change the version the user asks about, and don't copy changes across unless asked. Both share the same state/render architecture below; the v2-only parts are marked.

## Hard constraints (from the original spec — do not violate)

- **Code:** vanilla HTML/CSS/JS only. No frameworks, libraries, bundlers or npm. Keep each version in its single HTML file.
- **External resources:** none. No CDN scripts, web fonts or image files. Use the system font stack and inline SVG or Unicode glyphs for icons.
- **Storage:** none. No `localStorage`, `sessionStorage`, IndexedDB or cookies. Board state lives in memory and resets on refresh by design; a note in the header says so.
- **Banned calls and styles:** no `alert()`, no `confirm()` and no `!important`. Delete confirmation is an inline "Delete? Yes / No" row on the card.
- **Branding:** use the neutral "UOB IT PMO" text wordmark and a corporate blue palette. Do not use UOB's real logo or trademarks, or imitate any official UOB system.
- **Backend:** the only one is FormSubmit's AJAX JSON endpoint, set in the `FORMSUBMIT_ENDPOINT` config constant at the top of the script. Keep the placeholder address unless the user supplies one.

A quick compliance check:

```bash
grep -nE 'localStorage|sessionStorage|indexedDB|document\.cookie|alert\(|confirm\(|!important|<script src|<link' index.html v2/index.html
```

The command should print nothing.

## Architecture (inside the `<script>`)

- **Single source of truth:** `state = { tasks: [], filters: {} }`.
- **Transient UI state:** kept in a separate `ui` object:
  - `openMoveId` and `confirmDeleteId` track the open Move menu and the pending delete.
  - `focusKey` restores keyboard focus after a re-render.
- **Render cycle:** every change updates `state` or `ui` and then calls `renderBoard()`. It is the only function that writes card HTML: it runs `applyFilters()`, builds each column from `renderCard()` strings, and calls `renderSummary()`.
  - Do not change card DOM directly anywhere else.
  - Original: the header summary pills count all tasks. **v2:** the progress panel (headline, legend, overall stacked bar, one bar per project, table view) counts all tasks, while the column count badges count only the filtered tasks.
  - **v2:** the chart's DOM is built once by `buildChart()` so segment widths (`flex-grow` = count) can animate; `renderSummary()` only updates counts, widths, labels and `aria-*`. `wireChart()` adds the hover tooltip (text only), the project-row click that toggles the project filter, and a resize refit of the direct labels.
  - **v2:** status colours live in `--s-done`, `--s-progress`, `--s-blocked` and `--s-backlog`, applied through the `STATUS_CLASS` classes. They were validated as a colour-blind-safe set in the chart order `CHART_ORDER` (Done, In Progress, Blocked, Backlog); re-validate if you change them.
  - Focus comes back by matching `ui.focusKey` against the `data-focus-key` attributes.
- **Escaping:** every user-supplied value goes through `escapeHtml()` before it is inserted as HTML, including values used in attributes.
- **Actions:** `addTask()`, `moveTask()` and `deleteTask()` each change `state` and then re-render.
  - Task IDs are `UOB-ITPM-####` and come from the in-memory counter `nextId`.
  - The 8 seeded tasks use IDs 0001 to 0008.
- **Event handling:** card buttons use one delegated `click` listener on `#board`, routed by `data-action`. Native HTML5 drag and drop uses delegated listeners in `wireDragAndDrop()`; columns carry `data-status`, and the `.is-drop-target` class shows the highlight. The "Move ▸" menu is the keyboard fallback for drag and drop.
- **Add Task form:** `handleSubmit()` runs these steps in order:
  1. `validateForm()` checks the fields and shows inline errors.
  2. `addTask()` adds the card immediately (optimistic UI).
  3. `notifyNewTask()` sends the FormSubmit request inside try/catch. While it is in flight, the `inFlight` flag is set and the submit button shows "Sending…". A failure only shows a warning toast.
  4. The modal closes when the request settles.
  - `notifyNewTask()` has a timeout, and it treats a `{success:"false"}` JSON reply as a failure.
- **Dates:** compared as local `YYYY-MM-DD` strings (`todayISO()`, `addDays()`) to avoid timezone bugs. Seed due dates are relative to today, so the Overdue badges always show in the demo.
- **Reference lists:** `STATUSES`, `PROJECTS`, `CATEGORIES` and `PRIORITIES` fill the selects at init and are used for validation. A column's priority colour comes from `PRIORITY_CLASS` through a `--prio-color` CSS custom property.

## Content-Security-Policy (both versions)

Both files have a CSP `<meta>` tag and a `no-referrer` referrer policy: scripts are allowed only by the SHA-256 hash of the inline script, and `connect-src` allows only `https://formsubmit.co`. **Any edit to the `<script>` block changes its hash, and the page stops running until the hash is updated.** Recompute it after every script change, setting `p` to the file you edited (`index.html` or `v2/index.html`):

```bash
python3 -c "import re,hashlib,base64;p='v2/index.html';s=open(p).read();b=re.findall(r'<script>(.*?)</script>',s,re.S)[-1];h=base64.b64encode(hashlib.sha256(b.encode()).digest()).decode();open(p,'w').write(re.sub(r\"'sha256-[^']*'\",f\"'sha256-{h}'\",s,count=1));print(h)"
```

Then load the page and check the console for a CSP violation. If you change the FormSubmit endpoint's host, update `connect-src` too.

## Published artifact copy

The original design (`index.html`) is also published as a claude.ai artifact (https://claude.ai/artifact/7ErN67rTFWjTGLwfakpfjZ).

The artifact is built from a copy of `index.html` that differs in three ways:
- The `<!DOCTYPE>`, `<html>`, `<head>` and `<body>` wrapper tags are removed, because the artifact adds them itself.
- The tab title is shortened to "UOB IT PMO Kanban".
- `color-scheme: light` is added to `:root`.

The artifact's security policy blocks the FormSubmit `fetch`, so the email warning toast always appears there. This is expected.
