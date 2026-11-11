---
description: Show the connected Succession ONE principal and available orientation context.
---

Call `ping` without context first and report `workspace_name`,
`principal_type`, `principal.effective_role` and `principal.expires_at` (when
set).

- For `principal_type=workspace`, call `get_workspace_context` without context
  and summarise the date, integrations, and skill count.
- For `principal_type=agency` (agency operators only), call
  `get_agency_context` without context and summarise the Agency roster. Do not
  call a Workspace-scoped tool until a managed client has been resolved with
  `manage_managed_workspace` `action="resolve_workspace"`; then pass its
  returned `client_id` on every Workspace-scoped call.

If `ping` is unavailable, say the Succession ONE MCP server is not connected
and suggest `/so-connect`. Personal keys and connected apps are managed under
**Settings → API + MCP** in the web app.
