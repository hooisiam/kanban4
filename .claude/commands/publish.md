---
description: Security-scan the project, update README, GitHub About section and Pages workflow, then push to GitHub
argument-hint: <github-repo> [description]   e.g. hooisiam/kanban4 or https://github.com/hooisiam/kanban4
allowed-tools: Bash, Read, Edit, Write, Grep, Glob
---

# Publish this project to GitHub

Arguments: `$ARGUMENTS`

The first argument is the target GitHub repo, as `owner/repo` or a full `https://github.com/owner/repo(.git)` URL. Anything after it is an optional one-line description for the About section. If no repo is given, check `git remote get-url origin`; if that is also empty, **ask the user for the repo and stop**.

Work through the steps in order. Print a short status line as each one finishes. Stop and report if any step fails; do not skip ahead.

## 0. Preflight

- Run `gh auth status`. If it's not logged in, tell the user to run `gh auth login` themselves and stop. Never ask for, type or store a token.
- Normalise the repo to `OWNER/REPO`.
- Run `gh repo view OWNER/REPO`. If the repo doesn't exist, ask the user whether to create it (public or private) before running `gh repo create OWNER/REPO --<visibility> --source . --remote origin`.
- If `origin` is missing or points elsewhere, show the current value and ask before changing it (`git remote add origin …` / `git remote set-url origin …`).
- Note the current branch. The default publish branch is `main`.

## 1. Security scan (must pass before anything is uploaded)

Scan **every file that will be committed**: tracked files plus untracked files that aren't ignored (`git ls-files -co --exclude-standard`).

1. **Secrets.** Run `gitleaks detect --no-banner --redact -v` if it's installed (also `gitleaks detect --log-opts="--all"` to cover history). If it isn't installed, fall back to grep for:
   - private keys: `-----BEGIN [A-Z ]*PRIVATE KEY-----`
   - AWS keys `AKIA[0-9A-Z]{16}`, GitHub tokens `gh[pousr]_[A-Za-z0-9]{36,}` / `github_pat_`, Slack `xox[baprs]-`, Google `AIza[0-9A-Za-z_-]{35}`, OpenAI/Anthropic `sk-[A-Za-z0-9_-]{20,}` / `sk-ant-`
   - generic assignments: `(api[_-]?key|secret|token|passw(or)?d|client[_-]?secret)\s*[:=]\s*['"][^'"]{8,}`
   - Also run the same grep over `git log -p --all` so secrets in earlier commits are caught too.
2. **Unwanted files.** Flag `.env*`, `*.pem`, `*.key`, `*.p12`, `id_rsa*`, `*.sqlite`, `.DS_Store`, `node_modules/`, editor/OS junk, and any file over 5 MB. Offer to add them to `.gitignore` (create it if it's missing) and to `git rm --cached` any that are already tracked.
3. **Personal/internal data.** Grep for email addresses, phone numbers, internal hostnames/IPs (`10.`, `192.168.`, `172.16–31.`), and `localhost` URLs. The FormSubmit placeholder address in `FORMSUBMIT_ENDPOINT` is expected. Report anything else for the user to judge.
4. **Project compliance (from CLAUDE.md).** Run:
   ```bash
   grep -nE 'localStorage|sessionStorage|indexedDB|document\.cookie|alert\(|confirm\(|!important|<script src|<link' index.html
   ```
   It must print nothing.
5. **Client-side code risks** in `*.html`/`*.js`: `eval(`, `new Function(`, `setTimeout("` / `setInterval("` with string args, `document.write(`, and `innerHTML`/`insertAdjacentHTML` assignments whose interpolated values don't go through `escapeHtml()`. Also look for `http://` (non-HTTPS) resource URLs.
6. **Branding.** Confirm there's no real UOB logo, trademark image or imitation of an official UOB system (see CLAUDE.md).
7. **Workflow hygiene** in `.github/workflows/*.yml`: least-privilege `permissions:`, no secrets echoed, no `pull_request_target` running untrusted code.

Present the results as a table: check, result (PASS / WARN / FAIL), and details with `file:line`.
- **Any FAIL** (a real secret, a compliance hit, sensitive files about to be committed): stop, explain the fix, and don't push. If a secret is already in git history, tell the user to **rotate it** and that history must be rewritten; don't rewrite history yourself unless they ask.
- **Only WARNs**: list them and ask the user whether to continue.

## 2. README

Create or update `README.md` from what the code actually does (read `index.html` and `CLAUDE.md`; don't invent features). Keep any hand-written content the user already has, and update sections rather than overwrite them. Include:
- Title and a one-paragraph summary
- **Live demo** link: `https://OWNER.github.io/REPO/` (lower-case the owner)
- Features (short bullets)
- How to run it locally (open `index.html` in a browser; no build step)
- Tech/constraints (vanilla HTML/CSS/JS, single file, no storage, so state resets on refresh)
- Configuration (the `FORMSUBMIT_ENDPOINT` constant)
- Deployment (GitHub Pages via GitHub Actions, triggered on push to `main`)
- A disclaimer that it's an internal demo/training tool and not an official UOB system

## 3. GitHub Pages workflow

Make sure `.github/workflows/pages.yml` exists and deploys the static site with the official actions: `actions/checkout`, `actions/configure-pages`, `actions/upload-pages-artifact`, `actions/deploy-pages`, on `push` to `main` plus `workflow_dispatch`, with `permissions: contents: read, pages: write, id-token: write` and a `pages` concurrency group. Stage only the files the site needs (e.g. `index.html`) into `_site/`, never the whole repo. If the file already exists and meets this, leave it alone; otherwise, fix it. Use current major versions of the actions.

## 4. Commit and push

- Show `git status` and a `git diff --stat` summary.
- Stage only the intended files (never `git add -A` blindly; exclude anything flagged in step 1).
- Commit with a clear message ending with the Co-Authored-By attribution line required by the session.
- **Ask the user to confirm before pushing** and show them the target (`OWNER/REPO`, branch). Then run `git push -u origin <branch>`.
- Never force-push. If the push is rejected as non-fast-forward, run `git pull --rebase origin <branch>`, resolve any conflicts, rerun the security scan if new files came in, and ask again.

## 5. About section

Update the repository's About panel:

```bash
gh repo edit OWNER/REPO \
  --description "<description>" \
  --homepage "https://owner.github.io/REPO/" \
  --add-topic kanban --add-topic project-management --add-topic vanilla-js --add-topic github-pages
```

Use the description argument if one was given; otherwise write one sentence (≤ 350 chars) from the README summary. Show the description, homepage and topics before applying them, and keep any topics that are already there.

## 6. Enable Pages and verify the deploy

- Set Pages to build from GitHub Actions:
  ```bash
  gh api -X POST repos/OWNER/REPO/pages -f build_type=workflow \
    || gh api -X PUT repos/OWNER/REPO/pages -f build_type=workflow
  ```
- Find the latest `Deploy to GitHub Pages` run with `gh run list --workflow pages.yml -L 1` and wait for it with `gh run watch <id> --exit-status`.
- When it succeeds, `curl -sI https://owner.github.io/REPO/` and confirm HTTP 200 (a new site can take a minute to appear, so retry a few times).
- If it fails, show the failing step with `gh run view <id> --log-failed` and suggest a fix.

## 7. Report

Finish with a short summary: security scan result, files changed, commit SHA, repo URL, live Pages URL, and anything the user still needs to do (such as rotating a leaked key or reviewing WARNs).
