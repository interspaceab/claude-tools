# Todo conventions

This repo holds planning and todos for one organization. Code lives in the
code repos, never here. Every org repo uses the same format so that people and
Claude work the same way.

## Layout

```
CONVENTIONS.md
templates/item.md
projects/
  <project>/
    README.md                         # what it is, linked code repos, last_review
    backlog.md                        # loose ideas, one line each
    active/2026-11-23-invoice-sync.md
    done/2026/2026-11-02-api-auth.md
```

- **One file per work item.** An item is something to build or change that can
  be closed as a whole. Small steps are checkboxes inside it, not files.
- **File name:** `YYYY-MM-DD-<slug>.md`. Date = created; slug in kebab-case,
  max ~5 words. The file name never changes, so git history follows it.
- **Project names:** kebab-case, normally the code repo's name. An item can
  touch several code repos — list them in `repos`.
- **backlog.md:** `- [type] short description (YYYY-MM-DD)`. Promoted to a file
  in `active/` when work starts, and the line is removed.

### Project README

```markdown
---
last_review: 2026-11-25
---
# <project>
What it is, who owns it.

Code repos: acme/api, acme/web
```

## Lifecycle

| Status    | Location           | Meaning                                         |
|-----------|--------------------|-------------------------------------------------|
| `active`  | `active/`          | Planned or in progress                          |
| `blocked` | `active/`          | Waiting on something — set `blocked_by`         |
| `done`    | `done/<close year>/` | Finished, `## Outcome` filled in              |
| `dropped` | `done/<close year>/` | Not doing it — say why in `## Outcome`; set `duplicate_of` if merged |

Closing = set `status` and `closed`, fill in `## Outcome`, then
`git mv active/<file> done/<year>/<file>`. Never delete items — dropped work is
history too.

## Frontmatter

```yaml
title:        Short, descriptive title
type:         feat | fix | refactor | infra | docs | chore
status:       active | blocked | done | dropped
priority:     low | normal | high
created:      2026-11-23
due:          2026-12-15         # optional
effort:       s | m | l          # optional: s < half a day, m ≤ 3 days, l > 3 days
closed:                          # set on done/dropped
owner:        <user>
repos:        [acme/api]         # <owner>/<name> of affected code repos
tags:         [billing, fortnox] # 1–3, system/component names, lowercase
refs:         []                 # commits, PRs, tickets
blocked_by:                      # free text or link
duplicate_of:                    # slug of the item this was merged into
```

`type` follows conventional commits, matching commit messages. Filter with e.g.
`rg -l '^type: fix' projects/*/done/2026` or `rg -l 'tags:.*fortnox' projects`.

## Priorities

There are no manual rank numbers. Order comes from, in this order: blocked
items aside → `due` within 14 days or overdue → `priority` → `due` →
`created`. To move something up, raise `priority` or set `due`.

## Git

- `git pull --rebase` before reading or changing anything.
- One commit per logical change; push right after.
- Messages: `todo(<project>): add|update|close|drop|merge <slug>`
- Code commits that belong to an item include `todo: <slug>` in the message
  body, so reviews can find the evidence.
- Rebase conflict: resolve by hand, never overwrite other people's changes.
  Never force-push.

## Review

Run a review when `active/` passes ~15 items or `last_review` is older than 30
days: close what's finished, merge duplicates, drop what's stale. Proposals
need approval; nothing is closed automatically.

## Content

- **No secrets.** Passwords, tokens, keys → reference the vault.
- **No content across organizations.** Reference an id or code repo if needed;
  never copy text from another organization's todo repo.
- Write so a colleague understands the item without having been there.
- Templates, headings, frontmatter keys, slugs and commit messages are English.
  Free-text prose may follow the team's language.
