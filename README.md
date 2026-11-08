# Aoi Plugins

A marketplace catalog for [小葵（Aoi）](https://aoilang.com) plugins — portable [Agent Plugins](https://agent-plugins.org) for Claude Code, ChatGPT/Codex, and other MCP hosts.

## Plugins

| Plugin | Best for | What it does |
| --- | --- | --- |
| [English Cards](plugins/english-cards/README.md) (`aoi-grammar-cards`) | Claude Code, ChatGPT/Codex, Grok Bot, Cursor | Turns English grammar mistakes in chat into review cards and save to your Aoi account. |

## Install

### Claude Code

```text
/plugin marketplace add aoilang/aoi-plugins
/plugin install aoi-grammar-cards@aoi-plugins
```

### ChatGPT desktop / Codex

```bash
codex plugin marketplace add aoilang/aoi-plugins
```

Restart the ChatGPT desktop app, open the Plugins Directory, choose **Aoi Plugins**, and install **English Cards**. In Codex you can install from the CLI instead:

```bash
codex plugin add aoi-grammar-cards@aoi-plugins
```

Web ChatGPT has no repo-based install: add the plugin's MCP server or install the published listing. See the [English Cards README](plugins/english-cards/README.md).

### Direct folder install

Hosts that install a plugin folder directly can point at [`plugins/english-cards/`](plugins/english-cards/).

## Layout

```text
.claude-plugin/marketplace.json   catalog, read by Claude Code, ChatGPT desktop, and Codex
plugins/english-cards/            the English Cards plugin package
├── plugin.json                   portable Agent Plugins manifest
├── .claude-plugin/plugin.json    Claude Code manifest
├── mcp.json / .mcp.json          hosted MCP server
├── skills/aoi-grammar-cards/     when to save cards
└── assets/
```

## License

MIT — see [LICENSE](LICENSE).
