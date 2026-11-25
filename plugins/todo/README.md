# todo

A Claude Code plugin for planning work as todo items in **separate git repos per
organization**, kept outside your code repos.

Claude writes a plan before it builds, ticks tasks off as it goes, and closes the
item with an outcome when done — in a repo shared only with the people who work
in that organization (yourself, your company, an employer, a customer).

## Install

```
/plugin marketplace add interspaceab/claude-tools
/plugin install todo@interspace
```

## Commands

| Command | What it does |
|---|---|
| `/todo-next [repo] [--all]` | Prioritized list of active items touching a code repo, across all projects in its org repo. `--all` spans every org repo (shown only to you). |
| `/todo-review [project] [--org]` | Proposes closing finished items, merging duplicates and dropping stale ones, with evidence. Nothing changes without your approval. |

Planning, resuming, progress and closing happen through the `todo` skill as you
work — no command needed.

## How it works

```
~/todo/
  private/          your own todo repo (also holds map.yaml)
  acme/             one clone per organization
  employer/
  customer-example/
```

- **`map.yaml`** routes each code repo to an org and project, by git remote or
  working-dir path. First match wins. No match → Claude asks; it never guesses,
  because the wrong org means the wrong audience.
- **One file per work item** in `projects/<project>/active/`, moved with
  `git mv` to `done/<year>/` when closed. Low merge-conflict risk when several
  people and agents write at once.
- **Frontmatter** carries `type`, `status`, `priority`, `due`, `effort`, dates,
  `repos`, `tags` and refs. Claude searches frontmatter and reads full items
  only when needed, so thousands of items stay cheap.
- **Deterministic priorities:** blocked aside → due soon → priority → due →
  age. No manual rank numbers.
- **Duplicate check** before creating an item; **`todo: <slug>`** in code
  commits gives reviews hard evidence.
- **Pull before, push after**, one commit per change, never force-push.

See [templates/CONVENTIONS.md](templates/CONVENTIONS.md) for the full format.

## Set up

1. Create `~/todo/private/map.yaml` (or `~/todo/map.yaml`, or set `TODO_MAP`)
   from [templates/map.example.yaml](templates/map.example.yaml).
2. Clone existing org todo repos under `~/todo/`, or ask Claude to
   *"set up a new todo repo for <org>"* — it scaffolds README, CONVENTIONS and
   templates, and creates a private GitHub repo only if you ask.
3. Optional, to make the flow reliable: add to `~/.claude/CLAUDE.md`

   ```
   Before implementing non-trivial work, plan it with the todo skill. Never write todos in the code repo.
   ```

## Environment

| Variable    | Default                                           |
|-------------|---------------------------------------------------|
| `TODO_ROOT` | `root` in the map, else `~/todo`                  |
| `TODO_MAP`  | `<root>/private/map.yaml`, else `<root>/map.yaml` |

## Changelog

- **0.2.0** — `/todo-next` and `/todo-review`; `due`, `effort`, `duplicate_of`
  fields; required tags; duplicate check on create; frontmatter-first search;
  `last_review` per project; English templates.
- **0.1.0** — Initial release.
