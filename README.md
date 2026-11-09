# Aoi Plugins

**中文** | [English](README.en.md)

[小葵（Aoi）](https://aoilang.com) 的插件市场：为 Claude Code、ChatGPT/Codex 以及其他 MCP 客户端提供可移植的 [Agent Plugins](https://agent-plugins.org)。

## 插件

| 插件 | 适用 | 作用 |
| --- | --- | --- |
| [English Cards](plugins/english-cards/README.md)（`aoi-grammar-cards`） | Claude Code、ChatGPT/Codex、Grok Bot、Cursor | 自动检测和AI对话时的英文语法问题，并保存成复习卡片 |

## 预览
 ChatGPT 侧边栏展示所有卡片
![ChatGPT 侧边栏里展开的 English Cards 卡组](plugins/english-cards/docs/screenshots/chatgpt.png)



## 安装

### 优先自动安装，复制粘贴到你的AI Agent

```text
从 aoilang/aoi-plugins 安装 aoi-grammar-cards 插件
```

### Claude Code

```text
/plugin marketplace add aoilang/aoi-plugins
/plugin install aoi-grammar-cards@aoi-plugins
```

### ChatGPT 桌面端 / Codex

```bash
codex plugin marketplace add aoilang/aoi-plugins
codex plugin add aoi-grammar-cards@aoi-plugins
```

## MCP 服务器

默认安装完后会自动跳转到浏览器完成OAuth验证，如果没有跳转，需要手动添加MCP服务器,并完成登录验证

```text
https://aoilang.com/grammar/mcp/
```


## 许可

MIT — 见 [LICENSE](LICENSE)。
