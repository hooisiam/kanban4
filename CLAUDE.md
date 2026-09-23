# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A single-file IT Project Management Kanban board, labelled "UOB IT PMO". It is an internal demo/training tool. Everything lives in `index.html`: the markup, one `<style>` block and one `<script>` block. There is no build, lint or test tooling. To run it, open `index.html` directly in a browser (`file://`); it needs no server.

## Hard constraints (from the original spec — do not violate)

- **Code:** vanilla HTML/CSS/JS only. No frameworks, libraries, bundlers or npm. Keep everything in the single `index.html`.
- **External resources:** none. No CDN scripts, web fonts or image files. Use the system font stack and inline SVG or Unicode glyphs for icons.
- **Storage:** none. No `localStorage`, `sessionStorage`, IndexedDB or cookies. Board state lives in memory and resets on refresh by design; a note in the header says so.
- **Banned calls and styles:** no `alert()`, no `confirm()` and no `!important`. Delete confirmation is an inline "Delete? Yes / No" row on the card.
- **Branding:** use the neutral "UOB IT PMO" text wordmark and a corporate blue palette. Do not use UOB's real logo or trademarks, or imitate any official UOB system.
- **Backend:** the only one is FormSubmit's AJAX JSON endpoint, set in the `FORMSUBMIT_ENDPOINT` config constant at the top of the script. Keep the placeholder address unless the user supplies one.

A quick compliance check:

```bash
grep -nE 'localStorage|sessionStorage|indexedDB|document\.cookie|alert\(|confirm\(|!important|<script src|<link' index.html
```

The command should print nothing.

## Architecture (inside the `<script>`)

- **Single source of truth:** `state = { tasks: [], filters: {} }`.
- **Transient UI state:** kept in a separate `ui` object:
  - `openMoveId` and `confirmDeleteId` track the open Move menu and the pending delete.
  - `focusKey` restores keyboard focus after a re-render.
- **Render cycle:** every change updates `state` or `ui` and then calls `renderBoard()`. It is the only function that writes card HTML: it runs `applyFilters()`, builds each column from `renderCard()` strings, and calls `renderSummary()`.
  - Do not change card DOM directly anywhere else.
  - The header summary counts all tasks, while the column count badges count only the filtered tasks.
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

## Published artifact copy

The board is also published as a claude.ai artifact (https://claude.ai/artifact/7ErN67rTFWjTGLwfakpfjZ).

The artifact is built from a copy of `index.html` that differs in three ways:
- The `<!DOCTYPE>`, `<html>`, `<head>` and `<body>` wrapper tags are removed, because the artifact adds them itself.
- The tab title is shortened to "UOB IT PMO Kanban".
- `color-scheme: light` is added to `:root`.

The artifact's security policy blocks the FormSubmit `fetch`, so the email warning toast always appears there. This is expected.
