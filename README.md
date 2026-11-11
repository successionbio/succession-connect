# Succession ONE plugins

Official Succession ONE plugin marketplace for Claude Code and Codex. Each
bundle includes the production Succession ONE MCP declaration, product-surface
guides, safe workspace routing, and browser OAuth. No API token or authorization
header is required.

The MCP server is always named `succession-one` and connects to
`https://mcp.succession.bio/one/mcp`.

## Claude Code

```bash
claude plugin marketplace add successionbio/succession-connect
claude plugin install succession-one@succession
```

Open `/mcp`, choose `succession-one`, and authenticate in your browser (or run
`/so-connect`). Sign in to Succession ONE and approve the Workspace shown on the
approval page; each sign-in is bound to exactly one Workspace. Restart Claude
Code if prompted, then run `/so-whoami` (it calls `ping` and shows the
connected Workspace and role).

## Claude (claude.ai and Claude Desktop)

Add a custom connector with the URL `https://mcp.succession.bio/one/mcp`, then
sign in and approve the Workspace in the browser. No plugin is needed.

## Codex

```bash
codex plugin marketplace add successionbio/succession-connect
codex plugin add succession-one@succession
codex mcp login succession-one
```

Complete the Succession ONE approval page in your browser, then ask Codex to
call `ping` through `succession-one`.

## Personal keys

Clients that cannot open a browser can use a personal key from
**Settings → API + MCP** in the Succession ONE web app, sent as
`Authorization: Bearer <key>`. A key is scoped to the Workspace that was active
when it was created, and it can be revoked there.

## Moving from 0.1.x

Version 0.1.x pointed at an address that no longer serves MCP. After updating to
0.2.0, sign in again: in Claude Code open `/mcp`, clear the authentication for
`succession-one` and authenticate; in Codex run `codex mcp login succession-one`.
Sign-ins made before the move to `mcp.succession.bio` are not carried over.

## What is included

- Claude Code: surface-guide skills, bootstrap commands (`/so-connect`,
  `/so-whoami`, `/so-help`), and the production MCP declaration.
  Workspace-specific skills are discovered live through MCP.
- Codex: surface-guide skills, platform skill wrappers, live skill discovery,
  and the production MCP declaration.
- The `succession-agency` guide is for agency operators only. In a customer
  Workspace it says so and stops.

## Updates

Refresh the `succession` marketplace and update `succession-one` using your
client's plugin manager. Plugin releases follow semantic versioning. See
[CHANGELOG.md](CHANGELOG.md).
