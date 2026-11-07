# Aoi Grammar Cards

**English** | [中文](README.cn.md)

Turn English grammar mistakes in chat into study cards on your [小葵（Aoi）](https://aoilang.com) account. Compatible with ChatGPT, Grok Bot, Cursor, and other [Agent Plugins](https://agent-plugins.org) / MCP hosts.

While you chat in English, the model notices useful grammar issues, saves them as cards, and can show an interactive deck in the conversation. The same cards are also available on the web at [aoilang.com/grammar/review](https://aoilang.com/grammar/review/).

## What you get

- **Auto-capture** — meaningful grammar issues from your messages become cards (typos are ignored)
- **Grouped by sentence** — each card group shows the corrected sentence, then the pattern, rule, and example
- **Interactive deck** — search and review cards in the chat panel (where the host supports MCP Apps)
- **Synced to Aoi** — cards live on your Aoi account, not only in one chat

Inside a correction, removed words use `~~strikethrough~~` and replacements use `**bold**`.

## Install

You need a free [Aoi](https://aoilang.com) account. The first tool call opens OAuth so you can sign in.

Prefer installing **this folder as a plugin** when the host supports Agent Plugins: that loads [`skills/aoi-grammar-cards`](skills/aoi-grammar-cards/SKILL.md), which tells the model *when* to save cards. If you only add the MCP URL, the server still sends usage instructions, but many hosts ignore them — paste the [custom instructions](#make-cards-appear-reliably) below.

### ChatGPT

1. Open **Plugins → Add custom MCP server** (developer mode).
2. Enter `https://aoilang.com/grammar/mcp`.
3. Create as a plugin and finish **OAuth**.
4. Add the [custom instructions](#make-cards-appear-reliably) so cards save on every English message.

### Grok Bot / Codex / other Agent Plugins hosts

Install this folder as a plugin (recommended), or add the MCP URL `https://aoilang.com/grammar/mcp`. Authenticate when prompted, then attach the plugin with `@` if your host requires it. With MCP-only setup, also add the [custom instructions](#make-cards-appear-reliably).

### Cursor

Add a custom MCP server pointing at `https://aoilang.com/grammar/mcp`, then complete OAuth. Add the [custom instructions](#make-cards-appear-reliably) (or a user rule with the same text). You can also ask “show english cards” or “list english cards”.

## Make cards appear reliably

Paste this into the host’s custom instructions / user rules (ChatGPT: **Settings → Personalization → Custom instructions**):

```text
Before you answer, check my English for clear grammar mistakes worth learning
(ignore typos and style). If you find any, call create_english_learning_cards
first with the corrections, then continue with my request. Do not call
show_english_learning_cards right after saving. If there is nothing to teach,
skip the grammar tools.
```

## Use

Chat in English as usual. When there is something worth learning, the model saves cards and can open the deck. You can also ask:

- “Open my grammar cards.”
- “Review the English corrections from this chat.”
- “Save the grammar corrections from my last message.”

Review everything on the web anytime: [aoilang.com/grammar/review](https://aoilang.com/grammar/review/).


## License

MIT — see [LICENSE](LICENSE).
