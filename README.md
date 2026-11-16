# claude-plugins

Personal Claude Code plugin marketplace.

```sh
claude plugin marketplace add kr0sette/claude-plugins
claude plugin install kr0sette-work@kr0sette
```

Update later:

```sh
claude plugin marketplace update kr0sette
claude plugin update kr0sette-work@kr0sette
```

## Plugins

| Plugin | Description |
| --- | --- |
| [kr0sette-work](plugins/kr0sette-work) | `/kr0sette-work:review` — code review (work only) |

## Releasing a change

Installs are cached by version. Bump `version` in the plugin's `plugin.json` AND its `marketplace.json` entry, or updates won't arrive.
