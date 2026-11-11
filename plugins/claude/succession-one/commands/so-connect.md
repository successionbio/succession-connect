---
description: Connect or reconnect the Succession ONE MCP server.
---

Help me connect the Succession ONE MCP server.

1. Leave unrelated MCP connections unchanged. Use only the plugin's
   `succession-one` connection at `https://mcp.succession.bio/one/mcp`.
2. Do not add a token, Authorization header, or environment-variable
   placeholder. Open `/mcp`, choose `succession-one`, and start browser
   authentication.
3. Ask me to sign in to Succession ONE in the browser. The approval page
   (`https://one.succession.bio/mcp/authorize`) binds the connection to exactly
   one Workspace: ask me to check the Workspace shown there before I approve.
4. If Claude Code requires a restart to expose the plugin or MCP tools, tell me
   to restart it. Then call `ping` without context through `succession-one`.
   Report `workspace_name`, `principal_type` and `principal.effective_role`,
   and follow its `next_step`. If `db_reachable` is false, say the platform is
   degraded. Do not claim the connection works before that tool call succeeds.
5. If an earlier connection now fails with `invalid_token` (for example after
   the server moved to `mcp.succession.bio`), clear the old authentication for
   `succession-one` in `/mcp` and sign in again. Existing sign-ins do not carry
   over to the new address.

To use another Workspace, sign in again and approve that Workspace. For a
headless client, a personal key from **Settings → API + MCP** in the web app
works as a Bearer token; it is scoped to the Workspace that was active when it
was created.

Never remove or rename unrelated MCP connections.
