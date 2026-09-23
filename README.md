# UOB IT PMO — Kanban Board

A single-file IT Project Management Kanban board for internal demos and training. Track IT project tasks across **Backlog → In Progress → Blocked → Done**, filter the board, and add, move or delete tasks. Everything runs in the browser with no build step, and there are no dependencies.

**Live demo:** https://hooisiam.github.io/kanban4/

## Screenshot

![UOB IT PMO Kanban board](docs/screenshot.png)

## Features

- Four columns (Backlog, In Progress, Blocked, Done) with a count badge on each
- Header summary of task totals by status, plus the number of overdue tasks
- Tasks have an ID (`UOB-ITPM-####`), title, description, project/workstream, category, assignee, priority and due date
- Colour-coded priority (Critical / High / Medium / Low) and **Overdue** badges
- Move cards by drag and drop, or with the keyboard-accessible **Move ▸** menu
- Inline "Delete? Yes / No" confirmation on the card, with no pop-up dialogs
- Filter by project, priority and assignee
- Add Task form with inline validation, plus an optional email notification through [FormSubmit](https://formsubmit.co)
- Comes with 8 sample tasks whose due dates are relative to today

## Run locally

Clone the repo and open `index.html` in any modern browser. You don't need a server or a build step.

```bash
git clone https://github.com/hooisiam/kanban4.git
open kanban4/index.html
```

## Tech and constraints

- Vanilla HTML, CSS and JavaScript in a single `index.html`, with no frameworks, libraries or CDN assets
- **No persistence:** board state is kept in memory only, so refreshing the page resets it to the sample data
- No `localStorage`, cookies, `alert()` or `confirm()`
- All user input is HTML-escaped before it's rendered

## Configuration

To get an email whenever a task is added, set the FormSubmit endpoint at the top of the `<script>` block in `index.html`:

```js
const FORMSUBMIT_ENDPOINT = "https://formsubmit.co/ajax/YOUR_EMAIL@example.com";
```

If the request fails, the task is still added and a warning toast appears.

## Deployment

A GitHub Actions workflow (`.github/workflows/pages.yml`) publishes `index.html` to GitHub Pages on every push to `main`. You can also run it by hand from the **Actions** tab. In the repository settings, set **Settings → Pages → Source** to **GitHub Actions**.

## Disclaimer

This is an internal demo/training tool. It's **not** an official UOB system and doesn't use any UOB logos or trademarks.
