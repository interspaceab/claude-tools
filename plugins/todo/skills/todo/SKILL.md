---
name: todo
description: Plan, track and close work as todo items in per-organization todo repos outside the code repo. Use before building anything non-trivial, when resuming work, when asked about todos/backlog/what's next, or when asked to set up a new todo repo.
---

# Todo repos

Todos never live in the code repo. Each organization has its own git repo of
todos, cloned under a shared root. The format is defined in `CONVENTIONS.md` at
the root of each org repo — read it once per session and follow it; it wins
over anything here if they differ.

This skill's own `templates/` folder (two levels up from this file:
`../../templates/`) holds the reference templates and `map.example.yaml`.

## 1. Resolve where todos go

1. **Root** = env `TODO_ROOT`, else `root` from the map, else `~/todo`
   (`%USERPROFILE%\todo` on Windows).
2. **Map** = env `TODO_MAP`, else `<root>/private/map.yaml`, else
   `<root>/map.yaml`. If none exists, stop and offer to create one from
   `map.example.yaml`.
3. Get the code repo's remote: `git remote get-url origin`. It may fail (no repo
   or no remote) — then use path matching only.
4. **Normalize** the remote: strip protocol, `user@`, port and trailing `.git`;
   replace `:` after the host with `/`.
   `git@github.com:acme/api.git` → `github.com/acme/api`.
5. Walk `rules` top to bottom; **first match wins**. `remote` matches the
   normalized remote; `path` matches the absolute working dir (forward slashes,
   `~` expanded, case-insensitive on Windows). `*` = one segment, `**` = any
   number of segments.
6. **Project** = the rule's `project`, else the last remote segment, else the
   working dir name — lowercased kebab-case.
7. **No match → ask the user which org and project.** Never guess and never
   default to an org: writing to the wrong org exposes content to the wrong
   people. Offer to add a rule to the map afterwards.
8. If the org clone doesn't exist locally, offer
   `git clone <orgs.<name>.remote> <root>/<orgs.<name>.path>`. Don't clone
   without confirmation.
9. If `projects/<project>/` doesn't exist, create it with a short `README.md`
   (what it is, linked code repos) and an empty `backlog.md`, and say so.

State the resolved target in one line before writing, e.g.
`todo → acme/projects/api`.

## 2. Sync discipline

- `git -C <org repo> pull --rebase` before reading or writing.
- One commit per logical change, then `git push`.
- Commit message: `todo(<project>): add|update|close|drop <slug>`.
- Rebase conflict, or push rejected after one retry → stop and report. Never
  force-push, never overwrite others' changes.

## 3. Operations

**Plan (before building).** For non-trivial work, create
`projects/<project>/active/<YYYY-MM-DD>-<slug>.md` from the org repo's
`templates/item.md` (fallback: this skill's `templates/item.md`) *before*
writing code: goal, background, checkbox tasks, `repos`. Show the plan and get
a go before implementing. Trivial one-step changes don't need an item.

**Resume.** When starting work in a repo, list `active/` for the project
(title, status, priority, open checkboxes). Ask which to continue if more than
one is relevant.

**Progress.** Tick checkboxes as tasks finish. Add dated lines under the notes
section for decisions that aren't obvious from the code. Commit at natural
checkpoints, not after every checkbox.

**Close.** Set `status: done` (or `dropped`) and `closed: <today>`, fill the
outcome section with what was done, put commit hashes/PRs in `refs`, move
leftovers to `backlog.md`. Then
`git mv active/<file> done/<closing year>/<file>` and commit. Never delete
items.

**Backlog.** Loose ideas go in `backlog.md` as
`- [type] description (YYYY-MM-DD)`. Promote to an item when work starts and
remove the line.

**Query.** "What's open / what did we do in October / all fixes this year" →
`rg`/grep over frontmatter (`status:`, `type:`, `closed:`) in the resolved org
repo. Only search other org repos if the user explicitly asks.

**Init a new org repo.** When asked to set one up:
1. Confirm org short name, local path under the root, and remote URL (if any).
2. Create the folder, `git init`, copy `README.md`, `CONVENTIONS.md` and
   `templates/item.md` from this skill's `templates/`, add `projects/.gitkeep`.
   Fill the README placeholders with what the user gave you.
3. Commit: `todo: init repo`.
4. Only create a remote (`gh repo create <org>/<name> --private --source . --push`)
   if the user asks. Todo repos are always private.
5. Add the org and a matching rule to the map, then show the diff.

## 4. Content rules

- No secrets (passwords, tokens, keys, connection strings) — reference the
  vault instead.
- Never copy content between org repos. Reference an id or code repo if needed.
- Write for a colleague who wasn't in the session.
- Prose follows the language of the org repo's existing content (or the user's
  language for a new repo). Frontmatter keys, slugs and commit messages are
  always English.
