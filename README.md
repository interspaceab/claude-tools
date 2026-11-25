# claude-tools

Claude Code plugins from [Interspace AB](https://interspace.se).

## Install

```
/plugin marketplace add interspaceab/claude-tools
```

Then install the plugins you want:

```
/plugin install <plugin>@interspace
```

Update with `/plugin marketplace update interspace`.

## Plugins

| Plugin | Description |
|--------|-------------|
| [todo](plugins/todo) | Plan, track, prioritize and close work as todo items in per-organization git repos, kept outside the code repo. |

## Layout

```
.claude-plugin/marketplace.json   marketplace manifest, lists plugins/*
plugins/<name>/
  .claude-plugin/plugin.json      plugin manifest
  skills/ commands/ hooks/ ...    plugin contents
  README.md
```

Each plugin is versioned independently in its `plugin.json`.

## License

MIT
