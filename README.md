# ClaudePluginHub — plugin recommender for Claude Code

Ask Claude Code which plugins fit your project. This plugin detects your project's tech
stack and recommends Claude Code plugins from the
[ClaudePluginHub directory](https://claudepluginhub.com) — with evidence for each pick,
explicit warnings for plugins that execute code on your machine, and one-command installs.

## Install

From the community marketplace (after it appears there):

```
/plugin marketplace add anthropics/claude-plugins-community
/plugin install claudepluginhub@claude-community
```

Or directly from this repository:

```
/plugin marketplace add ClaudePluginHub/claudepluginhub-plugin
/plugin install claudepluginhub@claudepluginhub
```

## Use

In any project:

```
/claudepluginhub:recommend
```

Claude detects the stack (Next.js, Django, Rust, …), fetches recommendations grouped per
stack, and offers to install the ones you pick. Claude also invokes the skill on its own
when you ask things like "which Claude Code plugins should I use for this project?".

## What each recommendation shows

- **Why it's recommended** — co-install lift, component reuse, or recent-install signals
  from the directory, never bare popularity claims.
- **What's inside** — commands, agents, skills, hooks, and other component types.
- **⚠ runs code** — flagged when a plugin ships hooks, MCP/LSP servers, executables,
  monitors, or workflows that execute on your machine, so you can decide before installing.
- **Verified by owner** — when the maintainer has claimed the listing.

## Privacy

Stack detection runs through the [`claudepluginhub` CLI](https://www.npmjs.com/package/claudepluginhub),
which reads only dependency **names** from local manifests and a fixed allowlist of
well-known config **filenames** (like `next.config`, `go.mod`). File contents never leave
your machine. Requests are anonymous.

## Requirements

- Node 18+ (the skill runs `npx claudepluginhub`)

## License

MIT
