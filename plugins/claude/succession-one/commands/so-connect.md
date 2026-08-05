---
description: Connect or reconnect the Succession ONE MCP server.
---

Help me connect the Succession ONE MCP server.

1. Leave unrelated MCP connections unchanged. Use only the plugin's
   `succession-one` connection at `https://one.succession.bio/api/mcp`.
2. Do not add a token, Authorization header, or environment-variable
   placeholder. Open `/mcp`, choose `succession-one`, and start browser
   authentication.
3. Ask me to sign in and approve the exact Workspace shown by Succession ONE.
4. If Claude Code requires a restart to expose the plugin or MCP tools, tell me
   to restart it. Then call `ping` without context through `succession-one`.
   Confirm the returned home Workspace and principal type, and follow its
   `next_step`. Do not claim the connection works before that tool call succeeds.

Never remove or rename unrelated MCP connections.
