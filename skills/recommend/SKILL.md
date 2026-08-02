---
description: Recommend Claude Code plugins for the current project based on its detected tech stack. Use when the user asks which Claude Code plugins, skills, or tools they should install for this project, asks for plugin recommendations, or wants to discover plugins that fit their stack.
allowed-tools: Bash, Read
---

# Recommend plugins for this project

Recommend Claude Code plugins matched to the current project's tech stack, using the
ClaudePluginHub recommender. Stack detection happens through the `claudepluginhub` CLI,
which reads ONLY local manifest dependency names and well-known config FILENAMES — never
file contents — and sends them to `https://claudepluginhub.com/api/recommend`.

## Step 1 — Get recommendations

Run from the project root (requires Node 18+; allow up to 2 minutes — the first run
downloads the CLI):

```bash
npx -y claudepluginhub@latest recommend --json --source plugin
```

If "$ARGUMENTS" names a subdirectory of this project, run the command there instead.

Failure handling — NEVER invent recommendations from your own knowledge:

- Exit 1 with "No project manifests found": tell the user to run from a project root
  (a directory with package.json, pyproject.toml, go.mod, Cargo.toml, or similar), or to
  browse https://claudepluginhub.com/tools/for-your-project instead.
- HTTP 429: the API is rate-limited per IP; ask the user to retry in a few minutes.
- Network or other errors: report the error and link
  https://claudepluginhub.com/tools/for-your-project as the fallback. Stop there.
- Older CLI versions ignore `--source plugin`; that is harmless.

## Step 2 — Present the results

The JSON shape is `{ stacks: string[], sections: [{ stack, results: [...] }] }`.
Each result has: `slug`, `name`, `description`, `rank`, `evidence`, `url`,
`componentTypes`, `runsCode`, `hasVerifiedOwner`, and
`install { command, maintainerOverride, claude: { addCommand, installCommand } | null }`.
Fields other than the first four may be absent on older servers — guard every read.

Present each stack section in rank order, compact — for every plugin:

- Name, linked to `https://claudepluginhub.com` + `url` when `url` is present.
- Its one-line description.
- What's inside, from `componentTypes` (e.g. "commands, skills").
- The strongest one or two evidence signals, only when clearly non-trivial:
  `evidence.lift` ≥ 1.5 → "frequently installed with <stack> stacks";
  `evidence.reuseCount` ≥ 2 → "components reused by N other plugins";
  `evidence.recentInstalls90d` ≥ 5 → "N recent installs".
  Omit weak or absent signals entirely. Never describe `starCount`/`installCount`
  numbers as "installs" — the platform counts copy-clicks, not confirmed installs.
- Append a **⚠ runs code** warning when `runsCode` is `true` (the plugin ships hooks,
  MCP servers, LSP servers, executables, monitors, or workflows that execute on the
  user's machine). Use ONLY the `runsCode` field for this — never infer it from
  `componentTypes`, which cannot represent executables.
- Note "verified by owner" when `hasVerifiedOwner` is `true`.

Close by asking which plugins (if any) the user wants to install. Recommend at most two
unless they ask for more.

## Step 3 — Install what the user selects

For each selected plugin:

1. If `install.maintainerOverride` is `true` or `install.claude` is `null`: show
   `install.command` for the user to run themselves. Do NOT execute it.
2. Otherwise run `install.claude.addCommand`, then `install.claude.installCommand`
   via Bash.
3. If the plugin had the runs-code warning, restate it and get explicit confirmation
   BEFORE running any install command for it.
4. After installing, tell the user to run `/reload-plugins` (or restart Claude Code)
   to activate, and that `claude plugin uninstall <name>` removes a plugin.

If an install command fails, report the exact error output — do not retry silently or
fall back to a different install method.
