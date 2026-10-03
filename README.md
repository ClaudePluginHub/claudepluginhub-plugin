# ClaudePluginHub — plugin search and recommendations for Claude Code

Find Claude Code plugins without leaving Claude Code. Search the
[ClaudePluginHub directory](https://claudepluginhub.com) — 80,000+ plugins and their skills,
agents, commands, hooks and MCP servers, indexed from thousands of marketplaces on GitHub — or
let Claude recommend plugins that fit your project's tech stack.

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

Search for something that does a specific job:

```
/claudepluginhub:find postgres migrations
```

Claude shows the matching plugins and components with links, and installs the one you pick.
It also searches on its own when you ask things like "is there a skill for reviewing PRs for
security issues?".

Get recommendations for the current project:

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

`find` sends only your search phrase to claudepluginhub.com, anonymously.

Stack detection runs through the [`claudepluginhub` CLI](https://www.npmjs.com/package/claudepluginhub),
which reads only dependency **names** from local manifests and a fixed allowlist of
well-known config **filenames** (like `next.config`, `go.mod`). File contents never leave
your machine. Requests are anonymous.

## Requirements

- `curl` for `find`; Node 18+ for `recommend` and installs (they run `npx claudepluginhub`)

## License

MIT
