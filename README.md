# Succession ONE plugins

Official Succession ONE plugin marketplace for Claude Code and Codex. Each
bundle includes the production Succession ONE MCP declaration, product-surface
guides, safe workspace routing, and browser OAuth. No API token or authorization
header is required.

## Claude Code

```bash
claude plugin marketplace add successionbio/succession-connect
claude plugin install succession-one@succession
```

Open `/mcp`, choose `succession-one`, and authenticate in your browser. Restart
Claude Code if prompted, then ask it to call `ping` through `succession-one`.

## Codex

```bash
codex plugin marketplace add successionbio/succession-connect
codex plugin add succession-one@succession
codex mcp login succession-one
```

Complete the Succession ONE approval page in your browser, then ask Codex to
call `ping` through `succession-one`.

## What is included

- Claude Code: surface-guide skills, bootstrap commands, and the production MCP
  declaration. Workspace-specific skills are discovered live through MCP.
- Codex: surface-guide skills, platform skill wrappers, live skill discovery,
  and the production MCP declaration.

The MCP server is always named `succession-one` and connects to
`https://one.succession.bio/api/mcp`.

## Updates

Refresh the `succession` marketplace and update `succession-one` using your
client's plugin manager. Plugin releases follow semantic versioning.
