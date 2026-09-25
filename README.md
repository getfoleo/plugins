# Foleo agent plugin

Publish and manage Markdown and eligible active HTML on [Foleo](https://foleo.app)
from your agent, through Foleo's remote MCP server. Installing adds **no
credential**: the first Foleo tool call opens an OAuth sign-in in your browser,
and you approve access there.

Plugin version 0.1.2.

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

**Just the MCP server, no skill** (any MCP client): add
`https://mcp.foleo.app/mcp` as a remote (Streamable HTTP) server.

## About this repository

This repository is generated from Foleo's source by
`packages/foleo-agent-plugin/scripts/build-mirror.mjs`. Do not edit it by hand:
every change is overwritten by the next release. `SOURCE.json` and each
commit's `Foleo-Source-Commit` trailer name the source it was built from.

Report problems at https://foleo.app.
