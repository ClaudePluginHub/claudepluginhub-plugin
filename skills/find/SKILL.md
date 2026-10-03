---
description: Search the ClaudePluginHub directory (80,000+ Claude Code plugins and their skills, agents, commands, hooks and MCP servers, across thousands of marketplaces) for something that does a specific job. Use when the user asks for a plugin, skill, agent, command, hook or MCP server that does X ("is there a skill for…", "find me a plugin that…", "any MCP server for…"), or when the /plugin Discover tab doesn't have it.
argument-hint: <what you need, e.g. "postgres migrations">
allowed-tools: Bash(curl -sS -G --max-time 30 https://www.claudepluginhub.com/api/search:*)
---

# Find plugins and components

Search the ClaudePluginHub directory for Claude Code plugins and components that match a need.

## Step 1 — Build the query

Use "$ARGUMENTS" as the query. If it is empty, turn the user's request into a short
description of the job (3–8 words, e.g. `review pull requests for security issues`).
Never put file contents, secrets, paths or project names in the query; it is sent to
claudepluginhub.com.

## Step 2 — Search

```bash
curl -sS -G --max-time 30 https://www.claudepluginhub.com/api/search \
  --data-urlencode "q=<query>" \
  --data-urlencode "surface=claude_code_plugin" \
  -w '\nHTTP %{http_code}\n'
```

Failure handling — NEVER invent results from your own knowledge:

- HTTP 429: anonymous search is limited per hour; tell the user to retry later or search at
  `https://www.claudepluginhub.com/tools/search?q=<url-encoded query>`.
- Any other non-200 or network error: report it and give that same search link. Stop there.

## Step 3 — Present the results

The JSON has `results: [{ type, data }]` and, when more matches exist, `premiumLimit:
{ hasMore, totalAvailable }`. Guard every field read; omit anything absent.

For each result, in the order returned, show one compact entry:

- **Name** (`data.displayName` or `data.name`) and its kind (`type`), linked to its page —
  base `https://www.claudepluginhub.com`:

  | `type` | Page |
  |---|---|
  | `plugin` | `/plugins/<data.slug>` |
  | `marketplace` | `/marketplaces/<data.slug>` |
  | `skill` / `command` / `agent` / `output-style` | `/<type>s/<data.representativePlugin.slug>/<data.fingerprint>` |
  | `hook` | `/hooks/<data.representativePlugin.slug>` |
  | `mcp` | `/mcp-servers/<data.fingerprint>` |
  | `lsp` | `/lsp-servers/<data.fingerprint>` |

  For a component type without `data.representativePlugin` (other than `mcp`/`lsp`), skip the link.
- The description, cut to one line.
- For a component, the plugin it ships in: `data.representativePlugin.name`.
- `stars` (plugins) or `totalStars` (components) only as "★ N on GitHub". Never call any
  number "installs"; the directory counts copy-clicks, not installs.

Then, if `premiumLimit.hasMore` is true, add one line:
"N more matches on ClaudePluginHub: https://www.claudepluginhub.com/tools/search?q=<url-encoded query>"
with N = `premiumLimit.totalAvailable` minus the number shown.

## Step 4 — Install, only if the user asks

Components install with the plugin that contains them. The install command is
`npx -y claudepluginhub p/<slug>` (requires Node 18+), where `<slug>` is `data.slug` for a
`plugin` and `data.representativePlugin.slug` for a component. Marketplaces, and MCP/LSP
servers without a `representativePlugin`, have no one-step install; send the user to the page.

Before running an install, say that plugins can ship hooks, MCP servers or executables that
run on the user's machine and suggest checking the page first. Run it only after the user
confirms.
