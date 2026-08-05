---
description: Show the connected Succession ONE principal and available orientation context.
---

Call `ping` without context first and report the home Workspace and
`principal_type`.

- For `principal_type=workspace`, call `get_workspace_context` without context
  and summarise the date, integrations, and skill count.
- For `principal_type=agency`, call `get_agency_context` without context and
  summarise the Agency roster. Do not call a Workspace-scoped tool until a
  managed client has been resolved, then pass its returned `context` on every
  Workspace-scoped call.

If `ping` is unavailable, say the Succession ONE MCP server is not connected
and point me to **Settings → MCP** in the web app.
