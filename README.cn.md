# 小葵语法卡片

[English](README.md) | **中文**

把对话里的英文语法问题变成学习卡片，保存在你的 [小葵（Aoi）](https://aoilang.com) 账号里。可用于 ChatGPT、Grok Bot、Cursor 以及其他支持 [Agent Plugins](https://agent-plugins.org) / MCP 的客户端。

用英文聊天时，模型会发现值得学习的语法问题、保存成卡片，并在对话里展示互动卡组。同一批卡片也可以在网页复习：[aoilang.com/grammar/review](https://aoilang.com/grammar/review/)。

## 功能

- **自动收集** — 有学习价值的语法问题会变成卡片（忽略笔误）
- **按句子分组** — 每组先看改正后的整句，再看语法点、规则和例句
- **互动卡组** — 在支持 MCP Apps 的客户端里可搜索、回顾
- **同步到小葵** — 卡片存在你的小葵账号，不只留在某一次对话里

改正里，删掉的词用 `~~删除线~~`，替换的词用 `**加粗**`。

## 安装

需要一个免费的 [小葵](https://aoilang.com) 账号。第一次调用工具时会弹出 OAuth，按提示登录即可。

优先把**本目录安装为插件**（支持 Agent Plugins 的客户端）：这样会加载 [`skills/aoi-grammar-cards`](skills/aoi-grammar-cards/SKILL.md)，模型才知道**什么时候**该存卡。如果只添加 MCP 地址，服务端也会下发用法说明，但很多客户端会忽略——请贴上下面的[自定义说明](#让卡片稳定出现)。

### ChatGPT

1. 打开 **Plugins → Add custom MCP server**（需开发者模式）。
2. 填入 `https://aoilang.com/grammar/mcp`。
3. 创建为插件并完成 **OAuth**。
4. 加上[自定义说明](#让卡片稳定出现)，才会在每条英文消息里自动存卡。

### Grok Bot / Codex / 其他 Agent Plugins 客户端

推荐把本目录安装为插件；也可以只添加 MCP 地址 `https://aoilang.com/grammar/mcp`。按提示完成授权；若客户端要求，用 `@` 挂上插件。仅添加 MCP 时，请同时加上[自定义说明](#让卡片稳定出现)。

### Cursor

添加自定义 MCP，地址为 `https://aoilang.com/grammar/mcp`，完成 OAuth。再添加[自定义说明](#让卡片稳定出现)（或写成一条 user rule）。也可以说「show english cards」或「list english cards」。

## 让卡片稳定出现

把下面这段贴进客户端的自定义说明 / 用户规则（ChatGPT：**Settings → Personalization → Custom instructions**）：

```text
Before you answer, check my English for clear grammar mistakes worth learning
(ignore typos and style). If you find any, call create_english_learning_cards
first with the corrections, then continue with my request. Do not call
show_english_learning_cards right after saving. If there is nothing to teach,
skip the grammar tools.
```

## 使用

正常用英文对话即可。有值得学的语法点时，模型会保存卡片并可打开卡组。你也可以直接说：

- 「Open my grammar cards.」
- 「Review the English corrections from this chat.」
- 「Save the grammar corrections from my last message.」

随时在网页复习全部卡片：[aoilang.com/grammar/review](https://aoilang.com/grammar/review/)。


## 许可

MIT — 见 [LICENSE](LICENSE)。
