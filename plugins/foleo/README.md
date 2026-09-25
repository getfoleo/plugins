# Foleo agent plugin

Publish and manage Markdown and eligible active HTML on [Foleo](https://foleo.app)
from your agent, through Foleo's remote MCP server. Installing adds **no
credential**: the first Foleo tool call opens an OAuth sign-in in your browser,
and you approve access there.

Plugin version 0.1.3.

## Install

**Claude Code**

```sh
claude plugin marketplace add getfoleo/plugins
claude plugin install foleo@foleo
```

**Codex**

```sh
codex plugin marketplace add getfoleo/plugins
codex plugin add foleo@foleo
```

**VS Code and GitHub Copilot CLI** read this repository as a marketplace:

```sh
copilot plugin marketplace add getfoleo/plugins
copilot plugin install foleo@foleo
```

In VS Code, add `getfoleo/plugins` to the `chat.plugins.marketplaces`
setting, or run **Chat: Install Plugin From Source**.

**Cursor** loads the plugin from `plugins/foleo` (a `.cursor-plugin` manifest).
Team admins can import this repository as a team marketplace.

**Just the skill**, for any agent that reads Agent Skills:

```sh
npx skills add getfoleo/plugins
```

**Just the MCP server** (claude.ai, ChatGPT, Grok, Hermes, and any other MCP
client): add `https://mcp.foleo.app/mcp` as a remote (Streamable HTTP) server
and sign in when asked.

## About this repository

This repository is generated from Foleo's source by
`packages/foleo-agent-plugin/scripts/build-mirror.mjs`. Do not edit it by hand:
every change is overwritten by the next release. `SOURCE.json` and each
commit's `Foleo-Source-Commit` trailer name the source it was built from.

Report problems at https://foleo.app.
