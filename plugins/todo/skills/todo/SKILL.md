---
name: todo
description: Plan, track, prioritize and close work as todo items in per-organization todo repos outside the code repo. Use before building anything non-trivial, when resuming work, when asked about todos, backlog, priorities or what's next, when tidying up todos, or when asked to set up a new todo repo.
---

# Todo repos

Todos never live in the code repo. Each organization has its own git repo of
todos, cloned under a shared root. The format is defined in `CONVENTIONS.md` at
the root of each org repo — read it once per session and follow it; it wins
over anything here if they differ.

This skill's `templates/` folder (two levels up from this file:
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
9. If `projects/<project>/` doesn't exist, create it with a `README.md` (from
   the template's project section: what it is, linked code repos,
   `last_review:`) and an empty `backlog.md`, and say so.

The **repo key** of a code repo is `<owner>/<name>` from the normalized remote
(or the dir name if there is no remote). It's what goes in an item's `repos:`.

State the resolved target in one line before writing, e.g.
`todo → acme/projects/api`.

## 2. Sync discipline

- `git -C <org repo> pull --rebase` before reading or writing.
- One commit per logical change, then `git push`.
- Commit message: `todo(<project>): add|update|close|drop|merge <slug>`.
- Rebase conflict, or push rejected after one retry → stop and report. Never
  force-push, never overwrite others' changes.

## 3. Reading efficiently

Org repos can hold thousands of items. Never read `done/` wholesale.

- **Search frontmatter first** with `rg` (title, tags, type, status, repos,
  dates), then print **one line per hit**: date, title, type, status, project.
- **Open full files only when needed**, at most a handful. More than ~20 hits →
  show the list and ask the user to narrow down.
- `active/` is small by design; reading its frontmatter for listing is fine.
- Only search other org repos if the user explicitly asks.

## 4. Operations

**Plan (before building).** For non-trivial work:
1. **Duplicate check:** search `active/` and `backlog.md` of the org repo for
   overlapping title words, tags or `repos`. If something similar exists, show
   it and ask whether to continue that item instead.
2. Create `projects/<project>/active/<YYYY-MM-DD>-<slug>.md` from the org
   repo's `templates/item.md` (fallback: this skill's) *before* writing code:
   goal, background, checkbox tasks, `repos` (repo keys), **1–3 `tags`**
   (system/component names, lowercase), `priority`, and `due`/`effort` if known
   or asked.
3. Show the plan and get a go before implementing. Trivial one-step changes
   don't need an item. If backlog lines cover the work, remove them.

**Resume.** When starting work in a repo, list active items whose `repos`
include the current repo key (one line each, open checkbox count). Ask which
to continue if more than one is relevant. If there are more than ~15 active
items in the project, or `last_review` in the project README is older than 30
days or missing, suggest `/todo-review` once.

**Progress.** Tick checkboxes as tasks finish. Add dated lines under
`## Decisions & notes` for decisions that aren't obvious from the code. Commit
at natural checkpoints, not after every checkbox. Code commits that belong to an
item include `todo: <slug>` in the message body.

**Close.** Set `status: done` (or `dropped`) and `closed: <today>`, fill
`## Outcome` with what was done, put commit hashes/PRs in `refs`, move leftovers
to `backlog.md`. Then `git mv active/<file> done/<closing year>/<file>` and
commit. Never delete items.

**Backlog.** Loose ideas go in `backlog.md` as
`- [type] description (YYYY-MM-DD)`. Promote to an item when work starts and
remove the line. When a backlog passes ~50 lines, suggest a pass: promote,
merge or strike lines older than a year.

**Prioritize (`/todo-next`).** See section 5.

**Review (`/todo-review`).** See section 6.

**Query.** "What's open / what did we do in October / all fixes this year" →
frontmatter search per section 3.

**Init a new org repo.** When asked to set one up:
1. Confirm org short name, local path under the root, and remote URL (if any).
2. Create the folder, `git init`, copy `README.md`, `CONVENTIONS.md` and
   `templates/item.md` from this skill's `templates/`, add `projects/.gitkeep`.
   Fill the README placeholders with what the user gave you.
3. Commit: `todo: init repo`.
4. Only create a remote (`gh repo create <org>/<name> --private --source . --push`)
   if the user asks. Todo repos are always private.
5. Add the org and a matching rule to the map, then show the diff.

## 5. Priority list (`/todo-next [repo] [--all]`)

**Scope.** Default: active items in the resolved org repo whose `repos` contain
the current repo key, across **all projects** in that org repo. With a repo
argument: that repo key (resolve its org via the map). With `--all`: every org
repo under the root — shown to the user only, never written anywhere.

Read frontmatter only. **Sort deterministically**, in this order:
1. `status: blocked` → removed from the ranking, listed separately with
   `blocked_by`.
2. `due` within 14 days or overdue → first.
3. `priority`: high, normal, low.
4. `due`: earliest first; missing `due` after dated items.
5. `created`: oldest first.

**Output:**

```
#  Prio    Due         Eff  Title                         Project
1  high    2026-11-28  s    Rotate API token for billing  billing
2  high    —           m    Invoice sync retries          billing
3  normal  2026-12-15  l    Time report reconciliation    billing
Blocked: 1  (cert-renewal – waiting for vendor)
```

Mark overdue `due` with `!`. After the table, Claude may add at most three
short observations, clearly separated (e.g. "X unblocks Y", "A and B overlap",
"high + s: quick win"). Observations never change the order.

**Reprioritizing.** "Put X on top" → raise `priority` and/or set `due` on that
item (never introduce rank numbers), commit, show the new list.

## 6. Review (`/todo-review [project|--org]`)

Default scope: the current project; `--org`: every project in the org repo.
Read active items' frontmatter and checkboxes, then propose one action per item
with its evidence:

| Proposal | Evidence |
|---|---|
| **Close** | All checkboxes ticked, or commits with `todo: <slug>` in the current code repo (`git log --grep "todo: <slug>"`) newer than the item's last change |
| **Merge** | Overlapping title, tags and repos with another active item. Keep the older; move the other's open tasks into it; close the other as `dropped` with `duplicate_of: <slug>` |
| **Drop?** | No change to the item (git log on the file) in 60+ days and no matching commits |
| **Keep** | Everything else (don't list individually; give a count) |

Only check commits in code repos available locally; say which repos were not
checked. Present the proposals as a numbered list and let the user approve per
line or all at once. **Never act without approval.** Apply one commit per
action, then set `last_review: <today>` in the project README(s) and commit.

## 7. Content rules

- No secrets (passwords, tokens, keys, connection strings) — reference the
  vault instead.
- Never copy content between org repos. Reference an id or code repo if needed.
- Write for a colleague who wasn't in the session.
- Templates, headings, frontmatter keys, slugs and commit messages are always
  English. Free-text prose follows the language of the org repo's existing
  items; for a new repo, English unless the user says otherwise.
