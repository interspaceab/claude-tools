# Todo – <ORGANIZATION>

Planning and todos for work within <ORGANIZATION>: *what* to do, *why*, and
what got done. Code lives in the code repos, never here.

This repo is private and shared only with people who work within
<ORGANIZATION>. Never add passwords, tokens or keys.

## Contents

```
projects/<project>/
  active/      planned and in-progress items, one file each
  done/<year>/ finished and dropped items, by closing year
  backlog.md   loose ideas, one line each
  README.md    what the project is, linked code repos
templates/item.md
CONVENTIONS.md
```

Format, statuses and commit rules are in [CONVENTIONS.md](CONVENTIONS.md).

## Getting started

### 1. Clone into a shared root

All todo repos are cloned side by side under the same root, so they don't
collide with each other or with code repos:

```bash
# Linux/macOS
git clone <REMOTE-URL> ~/todo/<SHORTNAME>

# Windows (PowerShell)
git clone <REMOTE-URL> "$env:USERPROFILE\todo\<SHORTNAME>"
```

### 2a. Working by hand

1. `git pull --rebase`
2. Copy `templates/item.md` to
   `projects/<project>/active/YYYY-MM-DD-<slug>.md` and fill it in.
3. Commit and push: `todo(<project>): add <slug>`
4. When done: fill in `## Outcome`, set `status: done` and `closed:`, then
   `git mv active/<file> done/<year>/<file>`.

### 2b. Working with Claude Code

Install the plugin:

```
/plugin marketplace add interspaceab/claude-tools
/plugin install todo@interspace
```

Then create `~/todo/map.yaml`, which tells Claude which todo repo and project
each code repo belongs to. Minimal version:

```yaml
version: 1
root: ~/todo
orgs:
  <SHORTNAME>:
    path: <SHORTNAME>
    remote: <REMOTE-URL>
rules:
  - match: { remote: "<GIT-HOST>/<ORG>/*" }
    org: <SHORTNAME>
```

More examples in
[map.example.yaml](https://github.com/interspaceab/claude-tools/blob/main/plugins/todo/templates/map.example.yaml).
If a code repo matches no rule, Claude asks where the item belongs. It never
guesses.

Useful commands:

| Command | What it does |
|---|---|
| `/todo-next` | Prioritized list of active items for the current repo |
| `/todo-review` | Proposes closing, merging or dropping items — you approve |

## Rules in short

- One file per item. Small steps are checkboxes.
- Pull before changing, push right after. Never force-push.
- No secrets, and no content copied from other organizations' todo repos.
- Write so a colleague understands the item without having been there.
