# claude-code-plugins

Personal Claude Code plugin marketplace (`robertn702`). Directory-based marketplace registered in `~/.claude/plugins/known_marketplaces.json`.

## Structure

```
.claude-plugin/
  marketplace.json     # Marketplace manifest — lists all plugins with name, version, source path
plugins/
  <plugin-name>/
    .claude-plugin/
      plugin.json      # Plugin metadata (name, version, description, author)
      lsp.json         # LSP server config (optional — only for language server plugins)
    commands/           # Slash commands (optional)
    skills/             # Skills (optional)
```

## Adding a Plugin

1. Create `plugins/<name>/.claude-plugin/plugin.json` with metadata
2. Add commands, skills, or LSP config as needed
3. Register in `.claude-plugin/marketplace.json` under the `plugins` array
4. Install with `claude plugin install <name>@robertn702 --scope project|user`

## Plugins

| Plugin | Description |
|---|---|
| `self-improve` | Meta-prompting tools for creating/improving commands, agents, and managing Claude memory |
| `tsgo-lsp` | TypeScript LSP powered by tsgo (TypeScript 7 native Go compiler). Requires `npm install -g @typescript/native-preview` |
