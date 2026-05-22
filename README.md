# Better Tools

Better Tools is a Codex plugin that observes supported tool calls through a `PreToolUse` hook and turns that local corpus into practical tooling advice.

The hook writes a capped daily JSONL spool under `$PLUGIN_DATA/events/` and malformed payloads under `$PLUGIN_DATA/errors/`. Raw JSONL is not intended to become model context. `scripts/better_tools.py` produces compact deduped summaries and `indexes/tool-index.json`, so thousands of duplicate tool calls collapse into counts, fingerprints, normalized command patterns, and a few examples.

## What It Bundles

- `plugins/plugin-better-tools/hooks/hooks.json`: registers the `PreToolUse` hook.
- `plugins/plugin-better-tools/hooks/capture_pre_tool_use.js`: low-risk capture hook that redacts common secret-like values, truncates large records, appends JSONL, and exits successfully without blocking tools.
- `plugins/plugin-better-tools/scripts/better_tools.py`: local analyzer for `doctor`, `index`, `summary`, `patterns`, `blindspots`, `agents-md`, `report`, and `export`.
- `plugins/plugin-better-tools/skills/better-tools/SKILL.md`: Codex skill for tool optimization, tool blindspot analysis, new tool proposals, and AGENTS.md guidance.

## Install From GitHub

Make sure plugin hooks are enabled in `~/.codex/config.toml`:

```toml
[features]
hooks = true
plugins = true
plugin_hooks = true
```

Add this repo as a Codex plugin marketplace, then install the plugin:

```bash
codex plugin marketplace add johnlindquist/plugin-better-tools
codex plugin add plugin-better-tools@plugin-better-tools
```

Restart Codex after installing or updating the plugin. Open `/plugins` to confirm `plugin-better-tools` is installed and enabled, then open `/hooks` to review/trust the bundled `PreToolUse` hook if Codex asks for hook trust.

## Install From A Local Checkout

```bash
git clone https://github.com/johnlindquist/plugin-better-tools.git
cd plugin-better-tools
codex plugin marketplace add "$PWD"
codex plugin add plugin-better-tools@plugin-better-tools
```

If you edit the plugin locally after installation, reinstall it so Codex refreshes the cached copy:

```bash
codex plugin remove plugin-better-tools@plugin-better-tools
codex plugin add plugin-better-tools@plugin-better-tools
```

## Use The Skill

In Codex, ask for the skill directly:

```text
$better-tools analyze recent tool usage and suggest AGENTS.md improvements
```

The skill runs the analyzer first and uses compact summaries rather than dumping raw tool arguments into the conversation.

## Analyzer Commands

From the installed plugin root or this checkout:

```bash
cd plugins/plugin-better-tools
python3 scripts/better_tools.py locate
python3 scripts/better_tools.py doctor
python3 scripts/better_tools.py index --days 30
python3 scripts/better_tools.py summary --days 30
python3 scripts/better_tools.py agents-md --days 30
```

`index` writes a deduped compact index to:

```text
$PLUGIN_DATA/indexes/tool-index.json
```

When running manually outside Codex, the analyzer locates data in this order:

1. `--data-dir`
2. `$PLUGIN_DATA`
3. `$CLAUDE_PLUGIN_DATA`
4. `~/.codex/plugin-data/plugin-better-tools/active-data-root.json`
5. `~/.codex/plugin-data/plugin-better-tools`

If the locator points at a deleted temporary directory, the analyzer ignores it and falls back to the stable home directory.

## Local Checks

```bash
cd plugins/plugin-better-tools
node --check hooks/capture_pre_tool_use.js
python3 -m py_compile scripts/*.py tests/*.py
python3 -m unittest discover -s tests
python3 scripts/better_tools.py doctor
```

## Notes

`PreToolUse` captures supported Bash, file-edit, and MCP tool calls. It does not observe every possible Codex tool path. Treat the corpus as local sensitive telemetry; the hook redacts common secret-like keys and command fragments, but recommendations should use summaries and fingerprints rather than raw arguments.
